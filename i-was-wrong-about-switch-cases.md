# i was wrong about switch cases

switch statements are typically helpful when you need to perform different tasks based on the value of a single variable. in such cases, using multiple if-else clauses can be inconvenient, making the switch statement a preferable alternative. most of you have probably encountered it numerous times.

recently, a debate ensued between me and my colleague about a switch case implementation i did in JavaScript. my implementation looked something like this:

```javascript
switch (num) {
    case 1:
        x();
        break;
    case 2:
        y();
        break;
    case 3:
        z();
        break;
    default:
        err();
}
```

## rethinking the solution

with my roots in operating systems and language construction, i had seen this kind of implementation multiple times. so for me, this wasn’t inherently a bad solution. since i thought she had some valid points i started to research and reconsider this solution. as a result i discovered a better way to implement it.

## a better approach

dependent on the value of ***num***, we run ***x***, ***y***, or ***z***. in JavaScript, the same logic can be expressed more elegantly with a simple lookup table in the form of an object:

```javascript
const cases = {
    1: x,
    2: y,
    3: z
};

if (cases[num]) {
    cases[num]();
}
```

## advantages

this object-based approach offers several advantages:
- the code is more readable
- the conditions are stored as part of an object, making it easy to reuse the code
- changing or adding new conditions is straightforward
- all conditions are centrally managed within an object

in short, both my colleague and i were "wrong" in the way we were arguing, but, this debate led me to discover a much more elegant solution. so, thank you to my colleague for the discussion that prompted this!

## when to use switch cases

in my defense, there are situations where switch cases actually are better to use, even in JavaScript:
- when performance is an issue
- when each token is distinct
- when the context matters

### example

```javascript
const tokens = [
	'dist1','dist2','dist3',
	'mix1','mix2', 'mix3'
]

let context = "";
for(let i=0, token; (token = tokens[i]); i++) { // unsafe, but oh well
	switch(token) {
		case 'dist1': 
			context = x(); 
			break;
		case 'dist2': 
			context = y(); 
			break;
		case 'dist3': 
			context = z(); 			
			break;
		case 'mix1':
		case 'mix2':
		case 'mix3': 
			context = mix(); 
			break;
		default:
			err();
			break;
	}
}
```

this would in fact outperform the implementation that would have to look like this:

```javascript
function x(context){ return context; }
function y(context){ return context; }
function z(context){ return context; }
 
const cases = {
    1: x,
    2: y,
    3: z
};
 
if (cases[num]) {
    context = cases[num](context);
}
```

(because of the copy of context in each call)
