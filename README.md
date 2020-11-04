

# XState and Angular

As [its official documentation](https://xstate.js.org/docs/about/concepts.html) says "XState is a library for creating, interpreting, and executing finite `state machines` and `statecharts`, as well as managing invocations of those machines as actors". It can be used by `javascript`, hence some of its libraries in general such as `React`, `Vue`,  etc. 

On this case I'm going to give an example of how to implement `XState` inside `Angular`, and trigger some event changes.



## First things first

A good approach about using `XState` is we have to think about the `state machine` first, so we should think about every scenario before start to code by modeling some [statecharts](https://statecharts.github.io/). `XState` offers a tool to visualize our state machine and test the different scenarios and create the script to configure it [on their website](https://xstate.js.org/viz/). Here, you can also simulate changing states and following the workflow.

![preview of xstate's visualizer](https://i.imgur.com/yRrXzqU.png)



## Installing XState

Once we have our `statechart` defined (hence our state machine), we can install `XState` on our project via `npm` by running on our terminal inside the project's folder the following command line:  

```n
npm install xstate --save 
```



## Creating a State Machine

Since `Angular` uses `TypeScript`, we should follow the [Using TypeScript](https://xstate.js.org/docs/guides/typescript.html#using-typescript) guide, so we should create and `Schema` in order to give our machine the structure we want. For this example, I am going to use the same file, but I highly encourage you to separate the `State Machine`, its `Schema` and the component triggering it on different files.



First, we must import the `Machine` and `interpret` items from `XState` library on our file as follows:

```typescript
import { Machine, interpret } from 'xstate';
```



#### Defining types

For this example, I am going to describe a series of states for a character's animation, which has three states: `STILL`, `WALKING` and `RUNNING`. So in order to create the `Type` for our `State Machine`, we should create its `Schema` first:

````typescript
// ...

interface CharacterSchema {
  states: {
    STILL: {
        states: {
            YAWNING: {};
            BLINKING: {};
            CHECKING_CELLPHONE: {};
      };
    };
    WALKING: {};
    RUNNING: {};
  };
}

// ...
````

Since we want to switch between several animations when the character is on the `STILL` state, we also define some states inside it: `YAWNING`, `BLINKING` and `CHECKING_CELLPHONE`.



Once we have our `schema` defined, let's define some `Events`, which are going to trigger the changes on our `State Machine`.  For example, the action we are going to use to change from `STILL` to `WALKING` which is going to be triggered by the action `NEXT`. Also we added a `TIMEOUT` event in order to change the states inside the `STILL` state.

````typescript
// ...

interface CharacterSchema {
  // ...
}


type CharacterEvent = 
| { type: 'NEXT' }
| { type: 'TIMEOUT' };

// ...
````



#### Creating a State Machine 

Now that we have on place our types, we can create a `State Machine`. In order to create a Machine, we must call the `Machine` item from `XState` library, and define three types for it: `Context`, `Schema` and `Events`, so we must place our types `CharacterSchema` and `CharacterEvent`  there. Inside of it, we place our previously created `State Machine`. 



Since I am using `Angular` I am going to place this machine inside my `Constructor`'s component as follows:

````typescript
// ...

export class MyAngularComponent {
	
    constructor(){	
        const characterMachine = Machine<any, CharacterSchema, CharacterEvent>({
            id: 'characterState',
            initial: 'STILL',
            states: {
                STILL: {
                    initial: 'BLINKING',
                    states: {
                        // ...
                    }
                    on: {
                        NEXT: 'WALKING'
                    }
                },
                WALKING: {
                    on: {
                        NEXT: 'RUNNING'
                    }
                },
                RUNNING: {
                    type: 'final'
                }
            }
        });    
	} 
    
    // ...
} 
````



#### Starting the State Machine (We are almost there, I promise!)

In order to track and send messages (trigger the action) to our `State Machine` , we should create a `service`, which should be created before our `constructor` in order to use it on our function we want to trigger the change of state. So first, we just create the variable and I'm going to just give it an `any` type (remember to always use the right type on `TypeScript`).

````typescript
export class MyAngularComponent {

    private characterStateService: any;

    constructor() {
        // ...
    }
    
    // ... 
}
````

Once our `state service` has been declared, we proceed to initialize it inside the `constructor`, just right after we defined our `state machine`.

````typescript
	const characterMachine = Machine<any, CharacterSchema, CharacterEvent>({
		// ...
	});
	
	// first we interpret use the 'interpret' item from 'xstate' and send our machine
	this.characterStateService = interpret(characterMachine);

	// initialize the machine
	this.characterStateService.start();

	// subscribe to changes
	this.characterStateService.subscribe(state => {
	    console.log(state.value);
	})
````

First we `interpret` our machine, this will help us to track changes, execute side-effects and more functionalities according to their [interpreter documentation](https://xstate.js.org/docs/guides/interpretation.html#interpreter).

Then we `start` our machine, which is just going to run it.

Finally we `subscribe` to the state machine in order to get changes whenever we change it, inside, we place a simple `console.log()`, but the idea is to have our logic there regards changing states.



#### Updating the State Machine

Seems like everything is on place, but how are we going to update the machine? 

First, we should create a function, I'm going to call it `myFunction`, that will get triggered by something,  it could be a click, a keyup, a timer, etc. Inside of it, we can use our `state service` and send a message to it, which is going to be the `event` we want to trigger. On this case, we are going to send a `NEXT` message, which is going to change the state of our machine from `STILL` to `WALKING`. 

By doing so we should write something like this:

````typescript
export class MyAngularComponent {

    // ...
    
    constructor() {
        // ...
    }
    
    myFunction = () => {
        this.characterStateService.send('NEXT');
    } 

}
````

You can check the change on the `console.log()` we previously added inside the `subscription`.

That's it! 
