### Table of Contents
1. It all started with a for-loop ...
2. Lets go slightly more fp!
3. What about time complexity?!
4. What about pipelines vs for loop performance
5. Lazy evaluation in javascript...help me ramda!
6. Final notes

#### It all started with a for-loop ...
##### How do we cope with problems
Omg, I am a programer and I need to search collection. We have all been there: having an array and some rules to find a specific item out of it, the old fashioned way...

```javascript
let items = [1,2,3,4,5,6,7,8];
const isEven = x => x%2 === 0;
const moreThan3 = x => x > 3;

for(let i=0; i< items.length; i++){
    if(isEven(arr[i]) && moreThan3(arr[i])){
        ......
        break
    }
};
```
Yes, it feels cool and fast but it is imperative. That means we write every step of the algorithm - even dictating the iteration process, and no one does this without solid reason anymore. One must concentrate on the business rules, not the iteration process...
That code is not good as it is hard to follow, hard to refactor and support when complexity grows in size.

#### Lets go slightly more fp!
##### Functional programming, but why?

> In computer science, function composition is an act or mechanism to combine simple functions to build more complicated ones

Functional programming has many benefits for our code. Sometimes it is hard to come up with fp solution but it is always worth the hassle as the code is easier to reason about and read... and maintain, refactor, debug, memoize and test.
Lets have another example: we have a task to get all the people names and transform them into uppercase with some prefixes.
The iterative solution would be something like this
##### Some context...
```javascript
let people=[{name:'Adam',age:18},{name:'Eve',age:22}]
let names =[]
const makeUpperCase = name => name.toUpperCase();
const addPrefix = name => "Dear, "+name;
const getName = person => person.name

```
..and here it is:
```javascript
for (let index = 0; index < people.length; index++) {
    const name = getName(people[index])
    const personUpperCase = makeUpperCase(name);
    const personUpperCaseWithPrefix = addPrefix(personUpperCase)
    names.push(personUpperCaseWithPrefix)

}
console.log(names)//[ 'Dear, ADAM', 'Dear, EVE' ]
```
That hurts a bit, isn't it ? A more elegant solution would be to use the build-in methods like .map()
```javascript

const names = people
                  .map(getName)
                  .map(makeUpperCase)
                  .map(addPrefix)//[ 'Dear, ADAM', 'Dear, EVE' ]
```
One thing we need to admit is that the code is already human readable. We got rid of the manual iteration process rules and the intermediate variables. Yet another thing to admit is the time complexity is 3 times slower... wait, what?!

### What about time complexity?!

Yes, we do 3 iteration instead of one (with for loop). We gained readability, yet we lost the performance fight..or maybe not.

We can still do the transformation in one iteration like this
```javascript
const names = people.map(person=>addPrefix(makeUpperCase(getName(person)))) console.log(names)//[ 'Dear, ADAM', 'Dear, EVE' ]           
```
Well, I wouldn't call this the best alternative as it is not very readable, to say the least. Yet, it does iterate once over the collection. In order to do better we need to have a look at the function pipelines. In theory, we need to apply a collection of functions onto the collection of items. This collection is wrapped around the compose function which meets up the following rule
```javascript
compose(f, g)(..args) === f(g(..args));
```
or simply said, we will use the build-in .reduce() method to pass through the collection of items over the collection of functions
```javascript
let compose = (...fns)=> fns.reduceRight((f,g)=> (...args)=>f(g(...args)))
```
So, finally the code would look like this
```javascript
const transformer = compose(
    getName,
    makeUpperCase,
    addPrefix, 
)
const names = people.map(transformer)//[ 'Dear, ADAM', 'Dear, EVE' ]
```
Isn't it beautiful ! We do it like a pro, the code is easier to follow and we built a complex behaviour out of simple function we could further reuse. And everything would be so simple if all the algorithms came down only to mapping data transformations...

#### What about pipelines vs for loop performance
Lets get back to our first example with the for loop. It wouldn't be serious to talk about performance if we dont measure. So lets see how many steps does it take to our simple for-loop algorithm in order to find the first item given the business  rules.

```javascript
let items = [1,2,3,4,5,6,7,8];
let count=0;

let isEven =e=>{
    count++;
    return e%2==0
}
let moreThan3 =e=>{
    count++;
    return e >3
}
for(let i=0; i< items.length; i++){
    if(isEven(arr[i]) && moreThan3(arr[i])){
       ...item found!
        break
    }
};
console.log(count)// 6
```
6 steps needed to find the searched value. Lets see the functional approach using the fp library [ramda](https://ramdajs.com/)
 where there are defined utils like .filter() and .take() which will make our job easier by using [pointfree](https://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/) style of programming.
```javascript
let count=0;
let findFirstItem = pipe(
   filter(isEven),
   filter(moreThan3),
   take(1)
);
findFirstItem(items);
console.log(count)//12
```
[Show me the code](https://ramdajs.com/repl/?v=0.27.0#?let%20count%3D0%3B%0A%0Alet%20isOdd%20%3De%3D%3E%7B%0A%20%20count%2B%2B%3B%0A%20%20return%20e%252%3D%3D0%0A%7D%3B%0A%0Alet%20moreThan3%20%3De%3D%3E%7B%0A%20%20%20%20count%2B%2B%3B%0A%20%20%20%20return%20e%20%3E3%0A%7D%3B%0A%0Alet%20findFirstItem%20%3D%20pipe%28%0Afilter%28isOdd%29%2C%0Afilter%28moreThan3%29%2C%0Atake%281%29%0A%29%3B%0A%0Aconsole.clear%28%29%3B%0A%0A%0Alet%20arr%20%3D%5B1%2C2%2C3%2C4%2C5%2C6%2C7%2C8%5D%3B%0A%0Alet%20result%20%3DfindFirstItem%28arr%29%3B%0Aconsole.log%28result%29%0Acount%0A%0A%0A%0A%0A)

Well, well... 12! One thing is for sure- it is not ramda js, it is us. Ramda js and functional programming concepts are really useful tools. Yet, one needs to know how to use them. Having said that, it is obvious we might have chosen the wrong approach to fight the for-loop performance.
It is also true that the modern browsers are fast enough and this loss is not a big deal. Thus, we could argue the readability and maintenance are more than good enough reasons to leave the code as it is, closing our eyes for the tiny performance issue. Ugh... no, we can do better, there must be a way!

##### What we did wrong?

We applied the collection of functions over the collection. Hmm...
##### How we did it wrong way?!
After the first filter we are building a new collection with items [2,4,6,8]. After the second filter we again built a new collection with items [4,6,8]. At last, we get the first value which takes 1 step.
##### What is he problem?
We iterate over items 6 and 8 over and over again even though we have found our answer: 4. Why dont we just stop here ?! In fact, we do several iterations as the pipeline includes not only data mapping transformations but filtering as well.
##### The solution
We need a way to let the collection of functions pass through every single item, not the whole collection of items. Thus, we could stop when we find the first item meeting the restrictions.
##### The implementation
OMG, transducers !

#### Lazy evaluation in javascript...help me ramda!
Transducers are reducing functions. A reducing function is any function that can be passed to .reduce(). Ramda library has 2 functions which implement transducer protocol: into and transduce which enables us to do what we actually achieve in the for-loop, yet in fp way.
##### What if we..
What if we could take the first item and check if it satisfy all the next filter predicates. If it doesn't, we take the second and so on until we take an item which pass through all the filters and then we just stop iterating over the collection items.
#####T ransducers, here they are
That is exactly what they do - taking item by item and applying all the functions on them.
The functions are applied consecutively onto the items of the array. Thus, every item would go to the next filter only if it passed the previous filter predicate. Thus, we could apply the next filter predicate on the same item, without needing to wait the whole collection to be iterated over.

```javascript
let count=0;
let transducer = compose(
   filter(isEven),
   filter(moreThan3),
   take(1)
);

let result = R.into([], transducer, items); 
console.log(result)// [4]
count //6
```

[Show me the code](https://ramdajs.com/repl/?v=0.27.0#?let%20count%3D0%3B%0A%0Alet%20isOdd%20%3De%3D%3E%7B%0A%20%20count%2B%2B%3B%0A%20%20return%20e%252%3D%3D0%0A%7D%3B%0A%0Alet%20moreThan3%20%3De%3D%3E%7B%0A%20%20%20%20count%2B%2B%3B%0A%20%20%20%20return%20e%20%3E3%0A%7D%3B%0A%0Alet%20transducer%20%3D%20compose%28%0Afilter%28isOdd%29%2C%0Afilter%28moreThan3%29%2C%0Atake%281%29%0A%29%3B%0A%0Aconsole.clear%28%29%3B%0A%0A%0Alet%20arr%20%3D%5B1%2C2%2C3%2C4%2C5%2C6%2C7%2C8%5D%3B%0A%0Alet%20result%20%3D%20R.into%28%5B%5D%2C%20transducer%2C%20arr%29%3B%20%0Aconsole.log%28result%29%2F%2F%20%5B4%5D%0Acount%20%2F%2F6%0A%0A%0A%0A%0A)
In this context lazy evaluation represents the idea of not bothering to apply functions onto items when the goal of the function is achieved and those items that dont need processing.

#### Final notes

Well, yes...I didn't go into deep details about the transducers`s implementation and how they work. Generally, they are meant to be applied onto big collections and streams due to their processing nature. 
At the end of the day, we achieved great performance and level of abstraction that are needed to make our code more readable and maintainable, and last but not least- much more easier to reason about.
