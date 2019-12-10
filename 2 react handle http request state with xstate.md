# 2: react-handle-http-request-state-with-xstate

Instructor: [0:00](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Here I have a React app (`App.tsx`) with a single component whose purpose is to load a list of "Star Wars" characters and display them on the screen. Fetch people uses a mock network call to simulate latency and the chance that a network error will occur.

**App.tsx** 

    export interface Person {
    	name: string;
    	homeworld: string;
    }
    
    function App() {
    	const [isLoading, setIsLoading] = useState(fale);
    	const [results, setResults] = useState<Person[]>([]);
    	const [errorMessage, setErrorMessage] = useState('');
    	
    	function fetchData() {
    		setIsLoading(true);
    		fetchPeople()
    			.then(r => r.results)
    			.then (
    				res => {
    					setResults(res);
    					setIsLoading(false);
    				},
    				message => {
    					setErrorMessage(message);
    					setIsLoading(false);
    				}
    			);
    	}
    
    ... 

[0:16](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) In order to make this network call, I need three pieces of state, `isLoading`, `results`, and `errorMessage`. I need to be very careful to set these variables to specific values in order to display the correct UI for my app.

[0:31](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) When I start fetching data, I need to remember to **set** `isLoading` to **true** and then when I get some successful results back I need to **set the** `result` **value** and make sure to **set** `isLoading` **to** `false`. We also need to **set** `isLoading` **to** `false` ****in an `error` **state** and make sure to **set the** `errorMessage` ****to the network error.

[0:50](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) The rendered `HTML` looks like this. I have a `button` that, when you **click**, will `fetch` **the data**. When `isLoading` **is** `true`, it **sets a loading message**. When it's not loading and there's not an `errorMessage`, that means I have some **results** which I can **display**. If it's not loading but there is an `errorMessage`, I can **display my** `errorMessage`.

    function App() { 
    
    ... 
    
    return (
    	<div className="App"> 
    		<button onClick={() => fetchData()}>Fetch</button>
    		{ isLoading ? <p>Loading</p> : null }
    		{ !isLoading && !errorMessage ? (
    			<ul> 
    				{ results &&
    					results.map((person, index) => <li key={index}>{person.name}</li>)}
    				</ul>
    			} : null }
    		{ !isLoading && errorMessage ? <p>{errorMessage}</p> : null } 
    	);
    }

[1:10](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Even though I'm trying to be careful with how I use these variables in conjunction with each other, **there are still some bugs with my application. When I click Fetch, it loads the data successfully. When I click Fetch again, it shows an error.**

[1:28](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) However, when I click again, I got a successful response, but I'm still showing the `errorMessage`. **That's because I forgot to set the** `errorMessage` **to an empty string when I got a successful response.** Now, after I get a successful response and then an error, and then a subsequent successful response, it displays correctly.

    		res => {
    			setResults(res);
    			setIsLoading(false);
    			setErrorMessage("");
    		}

[1:51](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) What if I mash this Fetch button? I just triggered seven network calls, and then the responses for **those network calls came back in a random order, and updated my UI in really unexpected ways.** Let's rewrite this code to use a state machine to help solve these problems.

[2:12](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) I'll **make a folder for my machines (`machines`), and make a fetchMachine file (`fetch.ts`),** `import` ****`Machine` **from** `XState`, and `export` **our** `fetchMachine`. **Machine has an** `ID`**, an** `initial` **state** -- we'll say our initial state is **`idle`** -- **and a list of** `states`**.**

**Fetch.ts**

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	id: 'fetch',
    	initial: 'idle',
    	states: {
    
    	}
    })

[2:38](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) The **first state is** `idle` ****when no network call has been made yet, `pending` when a network call is in progress, `failed` when there's an error that's been returned, and `successful` when a valid response has come back.

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	id: 'fetch',
    	initial: 'idle',
    	states: {
    		idle: {},
    		pending: {},
    		failed: {},
    		successful: {}
    	}
    })		

[2:52](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Now we need to **define what events can trigger a transition from one state to another**. When you are in the `idle` state and a `fetch` event happens, we want to transition to the `pending` state. When you're in the `pending` state and a `resolve` event happens, we want to transition to the `successful` state.

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	id: 'fetch',
    	initial: 'idle',
    	states: {
    		idle: {
    			on: {
    				FETCH: 'pending'
    			}
    		},
    		pending: {
    			on: {
    				RESOLVE: 'successful'
    			}
    		},
    		failed: {},
    		successful: {}
    	}
    })

[3:11](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) **If a** `reject` **event happens, we'll transition to the** `failed` **state.** When you're in `failed` and a `fetch` event happens, we'll transition to `pending`. When you're in `successful` and a `fetch` event happens, we'll also transition to `pending`. I've taken our state machine code and pasted it into the visualizer at [xstate.js.org/viz](http://xstate.js.org/viz).

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	id: 'fetch',
    	initial: 'idle',
    	states: {
    		idle: {
    			on: {
    				FETCH: 'pending'
    			}
    		},
    		pending: {
    			on: {
    				RESOLVE: 'successful',
    				REJECT: 'failed'
    			}
    		},
    		failed: {
    			on: {
    				FETCH: 'pending'
    			}
    		},
    		successful: {
    			on: {
    				FETCH: 'pending'
    			}
    		}
    	}
    })

[3:40](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Here we can get a visual of what our machine looks like and how the transitions connect our states together. We start here in the idle state, and then a fetch event transitions to the pending state. Resolve will transition to successful. Fetch will take us back to pending. Reject takes us to failed, and fetch will also take us back to pending.

[4:05](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) **It would also be helpful if the machine kept track of the data that was returned by the network call.** We'll store this data in the `context` object. We'll have `results` in the context object and an `error` message. We want to set the `results` when this transition happens. Let's change this transition from using a plain `string` to using an `object` here.

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	id: 'fetch',
    	initial: 'idle',
    	context {
    		results: undefined,
    		message: undefined 
    	},
    	states: {
    		idle: {
    			on: {
    				FETCH: 'pending'
    			}
    		},
    		pending: {
    			on: {
    				RESOLVE: {target: 'successful', action: ['setResults']},
    				REJECT: 'failed'
    			}
    		},
    		failed: {
    			on: {
    				FETCH: 'pending'
    			}
    		},
    		successful: {
    			on: {
    				FETCH: 'pending'
    			}
    		}
    	}
    })

[4:31](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) This object with a target key is synonymous with the plain string that we had there before, but the object allows us to add a few extra properties to this transition. We're going to fire off some `actions` at the same time as this transition happens. We'll use the `setResults` action. We'll define that action in the second object of the machine function.

[4:52](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) We'll add an `actions` key. We'll create the `setResults` key and use the built in `assign` action that comes with XState. We'll put a function that takes the previous `context`(`ctx`) and the `event` that made the transition, and then we need to return a new object that will get merged into the existing context.

[5:12](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) We want to set the `results` value to `event.results`. TypeScript is complaining here because XState thinks that this event only has a `type` property on it. We'll set the `event` to `any` here to bypass these restrictions for now and later we'll add in some typings.

    export const fetchMachine = Machine({
    ... 
    }, {
    	actions: {
    		setResults: assign((ctx, event: any) => ({
    			results: event.results
    		}))
    	}
    }); 

[5:34](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) When this `resolve` transition happens, the `results` property on the `resolve` event will be passed into this assignment `action` and get set on the `results` inside of context up here. Now let's do the same thing for the `reject` transition.

[5:50](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Make this an `object`, add an `action` on here of `setMessage` and then we'll add the `action` here, assign the `context`. We'll set the `message` property to the `event.message`. Now that we have some actions, we'll paste our machine code into the visualizer again and make sure to remove any TypeScript types and click Update.

    pending: {
    	on: {
    		RESOLVE: {target: 'successful', action: ['setResults']},
    		REJECT: {target: 'failed', action: ['setMessage']}
    		}

    export const fetchMachine = Machine({
    ... 
    }, {
    	actions: {
    		...
    		setMessage: assign((ctx, event: any) => ({
    			message: event.message 
    	}))
    }); 

[6:24](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Now you can see that the resolve and reject transitions specify what actions are being triggered during that transition. Our machine is great except for actually fetching any data from the network. Let's add that as an `action`. When we enter the `pending` state, we want to trigger the `fetchData` action.

    pending: {
    		entry: ['fetchData'],		
    	on: {
    		RESOLVE: {target: 'successful', action: ['setResults']},
    		REJECT: {target: 'failed', action: ['setMessage']}
    		}

[6:45](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) We'll set it to `fetchData` but this action is not going to be defined inside of the machine itself. We're going to define it inside the component where the machine is used. Let's go to the app component (`App.tsx`) and wire up this state machine. `Import` the `fetchMachine` and `import` from XState/React the `useMachine` hook.

**App.tsx**

    import { fetchMachine } from './machines/fetch'; 
    import { useMachine } from '@xstate/react';

[7:08](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Instead of all these `state` variables, we'll set up a `fetchState` and send to `fetchMachine`. We'll use the machine, `fetchMachine`. Now there's a second parameter here for the `useMachine` hook in which we can set the `actions` that we want the component to override.

    function App() { 
    	const [fetchState, sendToFetchMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => {
    						setIsLoading(true);
    		fetchPeople()
    			.then(r => r.results)
    			.then (
    				res => {
    					setResults(res);
    					setIsLoading(false);
    				},
    				message => {
    					setErrorMessage(message);
    					setIsLoading(false);
    				}
    			);
    			}
    		}
    	});
    	... 
    }

[7:34](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) We'll set to `fetchData` action here. It has access to the current context and the event that came through. We'll take this `fetchData` function here, take that code and put it in here. **We don't need to do set** `isLoading` **anymore. We don't need to do** `setErrorMessage`**.**

[7:58](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) We don't need to do set `isLoading`. Instead of `setResults`, we will use the `sendToFetchMachine` to send an event of type `resolve` with the `results` property. Similarly, for the `errorMessage`, we'll send an event to the `fetchMachine` with the type of `reject` and a `message` property.

    function App() { 
    	const [fetchState, sendToFetchMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => {
    		fetchPeople()
    			.then(r => r.results)
    			.then (
    				res => {
    					sendToFetchMachine({type: 'RESOLVE', results: res})
    				},
    				message => {
    					sendToFetchMachine({type: 'REJECT', message})
    				}
    			);
    			}
    		}
    	});
    	... 
    }

[8:28](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) **Already, we've gotten rid of the need to set the** `isLoading` **flag to** `false` **in both of these branches.** Down here for this button click, instead of calling a `fetchData` function, **we will just send an event to the** `fetchMachine` **of type** `fetch`.

[8:46](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Since we don't have the `isLoading` boolean, we need to use that `fetchState.matches` `pending` to display a loading message. This is when the `fetchState` matches the `successful` state. Here, instead of this combination of Booleans, we just say, "When the state matches the `failed` state." The results are not in this component.

    function App() { 
    
    ... 
    
    return (
    	<div className="App"> 
    		<button onClick={() => sendToFetchMachine({ type: 'FETCH' })}>Fetch</button>
    		{ fetchState.matches('pending') ? <p>Loading</p> : null }
    		{ fetchState.matches('successful') ? (
    			<ul> 
    				{ fetchState.context.results &&
    					fetchState.context.results.map((person, index) => <li key={index}>{person.name}</li>)}
    				</ul>
    			} : null }
    		{ fetchState.matches('failed') ? <p>{fetchState.context.message}</p> : null } 
    	);
    }

[9:17](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) We need to get those from the `stateMachine`, the `context.results` and the `context.message`. Here TypeScript is complaining that `results` could be `undefined`. Let's change this to an empty `array` and change our message to an empty `string`.

**Fetch.ts**

    export const fetchMachine = Machine({
    	id: 'fetch',
    	initial: 'idle',
    	context {
    		results: [],
    		message: '' 
    	},
    
    ... 
    
    }); 

[9:39](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) Since we're no longer using the `results` object as an indicator of what `state` we're in or the `message` object as an indicator of what `state` we're in, **this is safe to do**. While we're at it, we'll write down some TypeScript `definitions` to help the users of this machine.

[9:57](https://egghead.io/lessons/react-handle-http-request-state-with-xstate) The states `interface` lists out all the `states` available. We'll make a `type` for all the events that are available on the fetchMachine. The context `interface` tells us what the context looks like. Then we add in the context, the state interface, and the events here. 

    import { Machine } from 'xstate'; 
    
    interface FetchStates {
    	states: {
    		idle: {},
    		pending: {},
    		succesfsful: {},
    		failed: {}
    	}
    }
    
    type FetchMachineEvents = 
    	{ type: 'FETCH' }
    	{ type: 'RESOLVE', results: any[] }
    	{ type: 'REJECT', message: string } 
    
    interface FetchContext { 
    	results: any[],
    	message: string 
    }
    
    
    
    export const fetchMachine = Machine<FetchContext, FetchStates, FetchMachineEvents>({
    ... 
    }); 

[10:3](https://egghead.io/lessons/react-handle-http-request-state-with-xstate#t=634)2 Now on our app component we can see that the context knows it has either a `message` or `results` object. Also it knows that the `reject` event needs to have a `message` property on it. Let's see this app in action now. Now we can click Fetch. It loads the data. Fetch again, shows an error. If you click the Fetch button lots of times, it only makes one network call and one successful return.

**App.tsx** 

    function App() {
    	actions: { 
    		... 
    		.then ( 
    			... 
    			message => {
    				sendToFetchMachine({type: 'REJECT', message}); 
    			}
    		); 
    	}
    });