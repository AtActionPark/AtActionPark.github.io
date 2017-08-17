---
layout: post
title: Implementing a neural net in javascript
---

What could go wrong?
<!--more-->

Since I've started to learn programming, I've always been fascinated by neural networks. So oviously I thought that would be a great projet to try. And I tried a lot, and consistently failed. I could implement some basic neuron structure, feedforward through a net, and all that good stuff, but the backprop algo was too hard for me to get, and the complete implementation, from loading datasets to being able to make good predictions, was always out of my reach.

But a few days ago I stumbled upon this fantastic [resource](http://neuralnetworksanddeeplearning.com)

And by fantastic I mean incredible. Everything explained from scratch, mathematics formulas, complete source code...
So whats the catch? well, no catch, except its, like usually in python, and uses lots of librairies.

I'm currently learning javascript, and being very stubborn, I feel like I can't understand something unless I write everything from scratch. So let's do it. No librairies, and not reusing some code I dont understand. It gonna be ugly but its gonna be my ugly code. Its gonna be extremely ineficient and slow, but oh well, that's a first step. Well not exactly, I'll allow myself to use 1 algo I dont understand, to create gaussian randoms, cause I'm not a mathematician.

So lets review the goal:
- Take the MNIST data
- Implement a simple network that can learn from it, and make predictions
- Draw numbers on a canvas, and see the magic happen

Nothing more fancy that what's exposed in the tutorial I linked. And it's gonna work this time!

The first good news is that if we look into the future, [it kinda does](http://atactionpark.github.io/NeuralTest/). Lets treat that as a proof of concept and rewind quite a lot.

Now, as a disclaimer, this is not a tutorial, this is not good code, nor a good implementation. This is ony a way for me to remember what went through my head at the time. If it can help someone struggling with a js implementation, thats cool, but there is a million of things to change.

So the first step is to enable matrix math. No idea whats the best way is, lets create a matrix class. I'll have a number of column and rows, and for each row, an array of dimension column. I kept the array of array in a separate value, called m, inside the matrix object.

```javascript
//Matrix definition. Only 2D
Matrix = function(column,row){
  var m = []
  this.column = column;
  this.row= row;

  for(var j = 0;j<row;j++){
    m[j] = []
    for(var i = 0;i<column;i++){
      m[j].push(0)
    }
  }
  this.m = m
}
```

Now that I have this, it's easy to create prototypes to allow for addition, dot product, scalar multiplication, hadamard product, transposition, cloning... I'll show the dot product for info, but that's pretty basic math.

```javascript
//returns the dot product of two matrix
Matrix.prototype.multiply = function(m1,m2){
  if(m1.column != m2.row){
    console.log("cant multiply matrices if colums m1 != rows m2")
  }
  var result = new Matrix(m2.column,m1.row)
  for(var j = 0;j<result.row;j++){
    for(var i = 0;i<result.column;i++){
      sum = 0
      for(var k = 0;k<m1.column;k++)
        sum += m1.m[j][k] * m2.m[k][i];
      result.m[j][i] = sum
    }
  }
  return result
}
```

So thats one thing taken care of.

I'm also gonna need few other helpers: 

One to transform an array into a matrix
```javascript
//Transforms an array to the used matrix format
function arrayToMatrix(arr){
  var m = new Matrix(arr.length,1)
    m.m[0] = arr
  return m
}
```

A zip function, to allow iteration on two lists at the same time.
```javascript
//Returns a list for iteration on two parameters
function zip() {
    var args = [].slice.call(arguments);
    var shortest = args.length==0 ? [] : args.reduce(function(a,b){
        return a.length<b.length ? a : b
    });

    return shortest.map(function(_,i){
        return args.map(function(array){return array[i]})
    });
}
```

A shuffle function
```javascript
function shuffle(o){
    for(var j, x, i = o.length; i; j = Math.floor(Math.random() * i),
     x = o[--i], o[i] = o[j], o[j] = x);
    return o;
}
```

Sigmoid and sigmoid prime functions

```javascript
function sigmoid(x){
  return 1.0/(1+Math.exp(-x))
}

function sigmoidPrime(x){
  return sigmoid(x)*(1-sigmoid(x))
}
```

Something to transform a result into an array
```javascript
//Transform the sample result (1,2,3 ...) into a 10 sized array
function labelToArray(l){
  var result = [0,0,0,0,0,0,0,0,0,0]
  result[l] = 1
  return result
}
```

And that should be it.

Now for the main course, the network class. It took me a long time to go through the material, step by step, doing the calculation by hand on small examples, making sure my results were consistent, and curling into a ball when they weren't. But thats thoe coder's life...

Once again I'm not explaining anything, please refer to the source for details

Here are the main functions:


```javascript
//Network definition. 
// 'sizes' represents the number of layers, and of neurons per layer
//Ex; [10,30,2] for a 3 layers net, with 10 inputs, 30hidden neurons,
// and 2 output
//the cost should be either QuadraticCost or CrossEntropyCost
Network = function(sizes, cost){
  this.numLayers = sizes.length;
  this.sizes = sizes;
  this.biases = []
  this.weights = []
  this.cost = cost
}
```

Network initialization. Nothing fancy, create empty biases and weights arrays of random matrixes
The input layer doesnt have biases, and theres one weight matrix between each two layers (so one less than the numbers of layers in total)
The weights can be initialized with a smaller distribution, as explained in the optimization chapter
```javascript
//initialize biases and weights
Network.prototype.init= function(){
  var biases = [];
  var weights = [];

  var noFirst = this.sizes.slice(1,this.sizes.length);

  //no biases for input
  noFirst.forEach(function(y){
    biases.push(new Matrix(y,1).randomize())
  })

  this.biases = biases

  var noLast = this.sizes.slice(0,this.sizes.length-1);
  var z = zip(noFirst,noLast)

  //weights are initialized with a smaller distribution
  z.forEach(function(t){
    weights.push(new Matrix(t[0],t[1]).randomize()
    .multiplyScalar(1.0/Math.sqrt(t[0])))
  })

  this.weights = weights
}
```

Feedforward take a matrix of inputs and calculate the result after passing through the network. I've defined an emtpy matrix, m, that I use for calculation purpose only. Nothing in it, just a dummy object for calling functions.
Also my matrix multiplications are in reverse because of the way I defined rows and columns

I assumed the format of my inputs would be something like [[input1,label1],[input2,label2]]
```javascript
//returns the output of the network for a given input
Network.prototype.feedforward = function(a){
  zip(this.biases,this.weights).forEach(function(t){
    var b = t[0]
    var w = t[1]
    
    a = m.add(m.multiply(a,w),b).applyAll(sigmoid)
  })
  return a
}
```

Well this is getting harder... Took me a while to understand what was going on here, and my implementation is pretty messy. I had some problems with array references and ended up cloning everything I was touching to make sure I didnt make mistakes. Went a bit overboard here...
```javascript
//Backprop algo. See http://neuralnetworksanddeeplearning.com/chap1.html
Network.prototype.backprop = function(x,y){
  var self = this
  nabla_b = []
  this.biases.forEach(function(b){
    nabla_b.push(m.clone(b).zero())
  })

  nabla_w = []
  this.weights.forEach(function(b){
    nabla_w.push(m.clone(b).zero())
  })

  activation = x;
  activations = [x]

  var zs = [];

  //feedforward
  zip(self.biases, self.weights).forEach(function(z){
    b = z[0]
    w = z[1]

    var z = m.add(m.multiply(activation,w),b)
    zs.push(z)

    activation = m.clone(z).applyAll(sigmoid)
    activations.push(activation)
  })
  
  //backward pass
  var delta = self.cost.delta(zs[zs.length-1],
                activations[activations.length-1],
                y)

  nabla_b[nabla_b.length-1] = delta
  nabla_w[nabla_w.length-1] = m.multiply(m.transpose(
          activations[activations.length-2]),
          delta)

  for(var i = 2;i<self.numLayers;i++){
    var z = zs[zs.length-i]
    var sp = m.clone(z).applyAll(sigmoidPrime)

    var delta1 = m.multiply(delta,
        m.transpose(self.weights[self.weights.length-i+1]))

    delta = m.hadamard(delta1,sp)

    nabla_b[nabla_b.length-i] = delta
    nabla_w[nabla_w.length-i] = m.multiply(m.transpose(
          activations[activations.length-i-1]),
          delta)
  }

  return [nabla_b,nabla_w]
}
```

Ok thats better now, just run the backprop algo and update the weights and biases accordingly
```javascript
//Backpropagates and update weights and biases for each batch of data
Network.prototype.updateMiniBatch = function(miniBatch,eta,lambda,n){
  var i = 0
  var self = this
  var nabla_b = []
  this.biases.forEach(function(b){
    nabla_b.push(m.clone(b).zero())
  })

  var nabla_w = []
  this.weights.forEach(function(b){
    nabla_w.push(m.clone(b).zero())
  })

  console.log("   Batch gradient descent")  

  miniBatch.forEach(function(d){
    var x = d[0]
    var y = d[1]

    var backprop = self.backprop(x,y)
    var delta_nabla_b = backprop[0]
    var delta_nabla_w = backprop[1]

    zip(nabla_b,delta_nabla_b).forEach(function(b,i){
      nabla_b[i] = m.add(b[0],b[1])
    })

    zip(nabla_w,delta_nabla_w).forEach(function(w,i){
      nabla_w[i] = m.add(w[0],w[1])
    })
  })

  var resultW = []
  zip(self.weights, nabla_w).forEach(function(w){
    var l2 = (1-eta*lambda/n)
    var newW = m.minus(m.clone(w[0]).multiplyScalar(l2),
        m.clone(w[1]).multiplyScalar(eta/miniBatch.length))
    resultW.push(newW)
  })
  var resultB = []
  zip(self.biases, nabla_b).forEach(function(b){
    resultB.push(m.minus(b[0],m.clone(b[1]).
        multiplyScalar(eta/miniBatch.length)))
  })

  self.weights = resultW
  self.biases = resultB
}
```

And run that stuff through multiple epochs, keeping track of what you want to keep track of
```javascript
//SGD algo. Batches the data, update each batch,
// and store epoch performance
Network.prototype.SGD = function(trainingData,
    epochs,
    miniBatchSize,
    eta,
    lambda,
    evaluationData,
    monitorEvaluationCost,
    monitorEvaluationAccuracy,
    monitorTrainingCost,
    monitorTrainingAccuracy){

  var self= this;
  evaluationCost = []
  evaluationAccuracy = []
  trainingCost = []
  trainingAccuracy = []

  if(evaluationData)
    var nData = evaluationData.length;
  var n = trainingData.length


  for(var j = 0;j<epochs;j++){
    trainingData = shuffle(trainingData);

    miniBatches = []
    for(var k = 0;k < n ; k+=miniBatchSize)
      miniBatches.push(trainingData.slice(k,k+miniBatchSize))

    console.log("Updating batches (" + miniBatches.length + ")")
    miniBatches.forEach(function(miniBatch){
      self.updateMiniBatch(miniBatch, eta, lambda, trainingData.length);
    })

    console.log("Epoch " + j +"'s training complete")
    if(monitorTrainingCost){
      cost = self.totalCost(trainingData, lambda)
      trainingCost.push(cost)
      console.log(" Cost on training data: " + cost)
    }
    if(monitorTrainingAccuracy){
      accuracy = self.accuracy(trainingData)
      trainingAccuracy.push(accuracy/n)
      console.log(" Accuracy on training data: " + accuracy/n)
    }
    if(monitorEvaluationCost){
      cost = self.totalCost(evaluationData,lambda)
      evaluationCost.push(cost)
      console.log(" Cost on evaluation data: " + cost)
    }
    if(monitorEvaluationAccuracy){
      accuracy = self.accuracy(evaluationData)
      evaluationAccuracy.push(accuracy/n)
      console.log(" Accuracy on evaluation data: " + accuracy/nData)
    }
    console.log(" ------ ")
    //eta*=0.9
  }

  return [evaluationCost,
    evaluationAccuracy,
    trainingCost,
    trainingAccuracy]
}
```

And some helpers:
Calculate network accuracy, comparing output with intput label
```javascript
//computes network accuracy. Feed forwards the data
// and compare to expected result
//returns the number of correctly identified points
Network.prototype.accuracy = function(data){
  var self = this
  
  var sum = 0
    data.forEach(function(d){
      var feed = self.feedforward(d[0])
      var i = feed.m[0].indexOf(Math.max.apply(Math, feed.m[0]));
      var j = d[1].m[0].indexOf(Math.max.apply(Math, d[1].m[0]));

    if(j == i)
      sum+=1
  })
    return sum
}
```

Compute cost
```javascript
Network.prototype.totalCost = function(data,lambda){
  var self = this
  cost = 0.0;
  data.forEach(function(d){
    a = self.feedforward(d[0])
    y = d[1]

    cost += self.cost.fn(a,y)/data.length
  })

  var sumNormSq = 0

  self.weights.forEach(function(w){
    var norm = w.norm()
    sumNormSq += norm*norm
  })
  cost += 0.5*(lambda/data.length)*sumNormSq
  return cost
}
```
And here's the cross entropy cost class

```javascript
CrossEntropyCost = function(){
}

CrossEntropyCost.prototype.fn = function(a,y){
  var result = 0;
  for(var j = 0;j<a.row;j++){
    for(var i = 0;i<a.column;i++){
      var aij = a.m[j][i]
      var yij = y.m[j][i]
      result += -yij*Math.log(aij) - (1-yij)*Math.log(1-aij)
    }
  }
  return result;
}

CrossEntropyCost.prototype.delta = function(z,a,y){
  return m.minus(a,y)
}
```


Well... Thats it i guess?

in order to try it I'll need some data. unfortunately the MNIST are in a weird format, and I didnt check what I would need before starting. So I'll need to build training sets from the MNIST file, and format it for my network to be able to read it.

I'll call my buildSet functions with a variable size arg, put the sorted data in a pixelValues array, and create a new array formated for my needs with formatSet()
I didnt spend too much time here, I know there's a much better way, but at that time I just wanted it to work...

```javascript
var dataFileBuffer;  
var labelFileBuffer;
var pixelValues = [];
var pixelValuesTest = [];

//Reads MNIST files and store set data in pixelValue
function buildTrainingSet(event){
  var size = event.data.trainingSize + event.data.validationSize
  for (var image = 0; image <= size; image++) { 
    var pixels = [];
    for (var y = 0; y <= 27; y++) {
        for (var x = 0; x <= 27; x++) {
          pixels.push(dataFileBuffer[(image*28*28)+(x+(y*28))+15]/255);
        }
    }
    console.log('Building Training Set')
    var output = JSON.stringify(labelFileBuffer[image + 8])
    var o = {input: arrayToMatrix(pixels),
         output: arrayToMatrix(labelToArray(output))}
    pixelValues.push(o)
  }
  console.log('Training Set built')
  return pixelValues
}

//Reads MNIST files and store set data in pixelValueTest
function buildTestingSet(event){
  var size = event.data.testSize
  for (var image = 0; image <= size; image++) { 
    var pixels = [];
    for (var y = 0; y <= 27; y++) {
        for (var x = 0; x <= 27; x++) {
          pixels.push(dataFileBuffer[(image*28*28)+(x+(y*28))+15]/255);
        }
    }
    console.log('Building Testing Set')
    var output = JSON.stringify(labelFileBuffer[image + 8])
    var o = {input: arrayToMatrix(pixels),
       output: arrayToMatrix(labelToArray(output))}
    pixelValuesTest.push(o)
 }
 console.log('Testing Set built')
 return pixelValuesTest
}

//Transform the set into needed format
function formatSet(set){
  var result  = [];
  for(var i = 0;i<set.length;i++){
    console.log('Formating set')

    result[i] = []
    result[i].push(set[i].input)
    result[i].push(set[i].output)
  } 
  return result
}
```

I went with a local file readers to pick the files and store them in buffers
```javascript
function readData(evt) {
    //Retrieve the first (and only!) File from the FileList object
    var f = evt.target.files[0]; 

    if (f) {
      var r = new FileReader();
      r.onload = function(e) { 
        var result = new Uint8ClampedArray(e.target.result)
        dataFileBuffer = result;
      }
      r.readAsArrayBuffer(f);
    } else { 
        alert("Failed to load file");
    }
}
function readLabel(evt) {
    //Retrieve the first (and only!) File from the FileList object
    var f = evt.target.files[0]; 

    if (f) {
      var r = new FileReader();
      r.onload = function(e) { 
        var result = new Uint8ClampedArray(e.target.result)
        labelFileBuffer = result;
      }
      r.readAsArrayBuffer(f);
    } else { 
        alert("Failed to load file");
    }
}
```

So now I can train my network:

Pick the training images and labels to fill the buffers, then build the sets (I used an event because the function is bound to a button in my case). The rest goes something like this

```javascript
//small values so it doesnt take ages 
var trainingSize = 6000
var testSize = 1000

trainingSet = pixelValues.slice(0,trainingSize)
testSet = pixelValuesTest.slice(0,testSize)
trainingData =  formatSet(trainingSet)
testData =  formatSet(testSet)

n = new Network(size,new CrossEntropyCost)
n.init()
n.SGD(trainingData,1,10,0.1,5.0,testData, true, true, true, true)
```
And it learns! After one epoch and such a small data set, the accuray is very low, but you can see it increasing after several epochs.
If I run the whole MNIST data through a few epochs and like 100 neurons in the hidden layer, I go up to aroune 96% accuracy. But it takes hours. But it works

To be perfectly honest, chrome doesnt like long running scripts, and I use chrome primarily, so I used a web worker to run the SGD. And I dont know how to fix it, but transfering the whole formated training data to the worker is a long running script in itself... So I can only transfer around 50000, and not the whole package. But web workers are new to me, I'll find a way...


So whats left is to create a drawable canvas, process it into a 28x28 grid, and feed it to the network. Well its a lot of code, and I'm lazy, so I wont detail it.


Woohoo!


Anyway, here's the [code](https://github.com/AtActionPark/NeuralNet)