---
layout: post
title: Image Evolver - Hill Climbing and simulated annealing.
---

Im taking a break from the Odin project. Is the procrastination finally taking over?

<!--more-->

Nope! I just learned how to use canvas and I cant wait to try some stuff with it. Ill get back to Odin as soon as I'm done.

So a couple years ago I stumbled on this [post](http://rogeralsing.com/2008/12/07/genetic-programming-evolution-of-mona-lisa/) by Roger Alsing, (and later [this one](http://alteredqualia.com/visualization/evolve/) by alteredqualia). I was learning C# so I thought it would be a good learning project, and I decided I' try to redo it. After a lot of tears, I ended up with something that worked, although very slowly, and not as good.

Now that I started to learn javascript and canvas, I wondered if I could make it better. Did it work?

Well, kind of. I took the same approach:

1. Create a polygon class, with a "dna" containing a series of genes representing it's characteristics (position, size, number of vertices, color, transparency).

2. Draw a set number of polygonson my canvas.

3. Slightly mutate one of the polygons.

4. Compare the result to the old result. If its better keep it, else, discard the changes and mutate another polygon.

5. Repeat

So the result is something like that:

{% highlight javascript %}
function hillClimbing(){
  rand = getRandomInt(0,population.population.length);
  population.mutate(rand);
  population.computeFitness();
  
  if (population.fitness > population.oldFitness)
    return;

  // Else discard the new solution and return to the previous one
  population.population[rand].revert();
  population.fitness = population.oldFitness;
}
{% endhighlight %}

The interesting part is the fitness function. Comparing the canvas to the original image. I decided to go with a per pixel normalized color comparison. Formula is following :

{% highlight javascript %}
function compareColorsNormalized(r1,g1,b1,r2,g2,b2){
var d = Math.sqrt((r2 - r1) * (r2 - r1) + (g2 - g1) * (g2 - g1) + (b2 - b1) * (b2 - b1));
var p = d / Math.sqrt(3 * 255 * 255);
return 100*p;
}
{% endhighlight %}

And the fitness computation function:

{% highlight javascript %}
Population.prototype.computeFitness = function(){
  this.draw();
  this.oldFitness = this.fitness;
  this.fitness = 0;
  imgd = canvas.getImageData(0,0,CANVAS_WIDTH,CANVAS_HEIGHT);
  data = imgd.data;

  for(var i = 0; i < data.length; i += 4) {
    this.fitness += compareColorsNormalized(baselineData[i],baselineData[i+1],baselineData[i+2],data[i],data[i+1],data[i+2]);
  }

  this.fitness /= CANVAS_WIDTH*CANVAS_HEIGHT;

  // 100 - fitness returns a number that goes from 0 (opposite of result) to 100 (equal to result)
  this.fitness = 100-this.fitness;
}
{% endhighlight %}

So this is the big one, the one that takes most of the computation time.
I obviously dont need to redraw all polygons, redrawing only the mutated one would be better but the real bottleneck is actually not there, its in the `getImageData()` function. I haven't found yet a solution there.

When all's said and done, the result is actually not bad:
![result](/images/clem.png)

I look a bit less pixelated in real life but thats pretty close.

But there's a problem : Local maximum. After some time, progress becomes extremely slow, and the algo can not try to look if better solutions are achievable by gong another route.

After some searching, I heard about simulated annealing. The method and implementation are actually quite simple.

The idea behind it is that we will let the algo make some mistakes, to ensure that it will be able to get out of local maxima. So it works like this:

1. Slightly mutate one of the polygons.

2. If the result is better than the old one, keep it.

3. If its not, we still may keep it, if the difference is not that huge (deltaE : difference between the old fitness and the new one), if its not too late in the process (temperature : variable that decreases with time), and if we are lucky (P: acceptance probability). So it goes something like this:

{% highlight javascript %}
function simulatedAnnealing(){
  rand = getRandomInt(0,population.population.length);
  population.mutate(rand);
  population.computeFitness();

  // Decrease temperature
  temperature *= alpha;
  
  // First Case : fitness > old fitness. Keep the solution
  if (population.fitness > population.oldFitness)
    return;

  // Second case : fitness < old fitness. 
  // Difference in fitness of the 2 solutions
  deltaE = population.oldFitness - population.fitness;

  // Acceptance probability
  // P is higher if deltaE is lower 
  // P gets lower and lower with temperature decrease
  P = Math.exp(-deltaE/temperature);
  
  // Keep the solution if a random(0,1) < P
  if(deltaE>0 && Math.random() < P){
    return;
  }

  // Discard the new solution and return to the previous one
  population.population[rand].revert();
  population.fitness = population.oldFitness;
}
{% endhighlight %}

The real one is a bit more complicated, as i try to keep track of the best population, allow te revert to it after a set number of steps, and do multiple pass for each temperature step, but thats the general idea.

So the real question is, does it work? Well I dont really know... As the process is random, very slow, and there are a lot of variables, I haven't really been able to get a definitive answer. 

Here is another test:
![result](/images/clem2.png)

There doesn't seem to be a huge difference in fitness, even after a few hours, but I dont have enough repetitions to really know. If you judge only on the Boba Fett shirt (most scientific method I know), this one is a clear winner though.

But still, it was fun.

EDIT: After a few more tries, I ended up adding a method for adding new polygons to the population after x unsuccessful attemps. It seems that the best results are obtained by starting with a low poly count, and progressively adding new ones when the simulation starts to slow down. Perhaps I should measure the fitness variation over time instead to decide when to add new polys.

Obligatory Mona Lisa result (180 polys): ![Mona result](/images/result.png)

If you squeeze your eyes real hard it's pretty good. It's still very far from the Roger Alsing's implementation. Any ideas on what I did wrong?

Anyway, here's [code](https://github.com/AtActionPark/image_evolve), and the [result](http://htmlpreview.github.io/?https://github.com/AtActionPark/hill_climbing_image/blob/master/index.html). 
