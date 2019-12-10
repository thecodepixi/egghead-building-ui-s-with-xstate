# 3: react-invoke-an-xstate-service-when-entering-a-state

Instructor: [0:00](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) Let's reuse this state machine to fetch planets as well as people. Changes name to `fetchPeople` state, `fetchPlanet` state. Let's change the network call down here to `fetchPlanets` instead of people. Now we'll duplicate the rendered HTML. Let's copy this. We'll update the person references to planet. Now we have two buttons. The top list loads people, and the bottom list loads planets.

**App.tsx**

    function App() { 
    	const [fetchPeopleState, sendToPeopleMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => {
    		fetchPeople()
    			.then(r => r.results)
    			.then (
    				res => {
    					sendToFetchMachine({ type: 'RESOLVE', results: res });
    				},
    				message => {
    					sendToFetchMachine({ type: 'REJECT', message });
    				}
    			);
    			}
    		}
    	});
    
    	const [fetchPlanetState, sendToPlanetMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => {
    		fetchPlanets()
    			.then(r => r.results)
    			.then (
    				res => {
    					sendToFetchMachine({ type: 'RESOLVE', results: res });
    				},
    				message => {
    					sendToFetchMachine({ type: 'REJECT', message });
    				}
    			);
    			}
    		}
    	});
    
    return (
    	<div className="App"> 
    		<button onClick={() => sendToPeopleMachine({ type: 'FETCH' })}>Fetch</button>
    		{ fetchPeopleState.matches('pending') ? <p>Loading</p> : null }
    		{ fetchPeopleState.matches('successful') ? (
    			<ul> 
    				{ fetchPeopleState.context.results &&
    					fetchPeopleState.context.results.map((person, index) => <li key={index}>{person.name}</li>)}
    				</ul>
    			} : null }
    		{ fetchPeopleState.matches('failed') ? <p>{fetchPeopleState.context.message}</p> : null } 
    	
    <hr> 
    
    <button onClick={() => sendToPlanetMachine({ type: 'FETCH' })}>Fetch</button>
    		{ fetchPlanetState.matches('pending') ? <p>Loading</p> : null }
    		{ fetchPlanetState.matches('successful') ? (
    			<ul> 
    				{ fetchPlanetState.context.results &&
    					fetchPlanetState.context.results.map((planet, index) => <li key={index}>{planet.name}</li>)}
    				</ul>
    			} : null }
    		{ fetchPlanetState.matches('failed') ? <p>{fetchPlanetState.context.message}</p> : null } 
    
    	);
    
    }

[0:33](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) It's great to be able to reuse this fetch machine logic, **but every time a component uses this fetch machine, they have to remember to send a resolve event and send a reject event** when the network call returns. If this logic will happen every single time, **it would be better if it was inside the machine itself.**

[0:53](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) **We can do this by having the `pending` state `invoke` a service.** We'll add the `invoke` key. We'll add the `invoke` key and give it an object that describes the service.

[1:03](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) A service has a source  (`src`) property in which we can define a service directly with a callback or a promise or we can give it a name for the service and allow the component to define the service itself. Then we can add an `onDone` transition, which will just be this resolve transition and an `onError` transition, which we will define as the reject transition. Now we don't need this action or these transitions.

**fetch.ts** 

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	... 
    
    	pending: {
    		invoke: {
    			src: 'fetchData',
    			onDone: { target: 'successful', actions: ['setResults'] },
    			onError: { target: 'failed', actions: ['setMessage'] }
    		}
    	}
    	...
    
    })

[1:29](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) **The** `onDone` **and** `onError` **events are generated automatically for us by X state.** They have their values put on a `data` property. We need to update our actions to look at that property. **Now** `fetchData` **is no longer an** `action`**, it's a service.**

    import { Machine } from 'xstate'; 
    
    export const fetchMachine = Machine({
    	...
    
    	actions: {
    		setResults: assign((ctx, event: any) => ({
    			results: event.data 
    		}));
    		setMessage: assign((ctx, event: any) => ({
    			message: event.data 
    		}));
    	}
    	... 
    
    })

[1:47](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) When we use this machine, we'll put `fetchData` under the services key and this service can return a promise and observable or even another machine. In our case, we'll return a promise and we no longer need to send these events to the machine because the `onDone` and `onError` transitions take care of this for us.

**App.tsx**

    function App() { 
    	const [fetchPeopleState, sendToPeopleMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => {
    				return fetchPeople()
    					.then(r => r.results) 
    			}
    		}
    	});
    
    	... 

[2:07](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) We'll do the same thing for the planet machine. We can shorten this. We'll update this one to the shorter version. Reusing this fetch machine is a lot simpler. All you worry about are the pieces that change the network call and then deciding how to render the result. Let's take a look at the app and you see that it's working correctly.

    function App() { 
    	const [fetchPeopleState, sendToPeopleMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => fetchPeople().then(r => r.results) 
    		}
    	});
    
    const [fetchPlanetState, sendToPlanetMachine] = useMachine(fetchMachine, {
    		actions: {
    			fetchData: (ctx, event) => fetchPlanets().then(r => r.results) 
    		}
    	});
    
    	... 

[2:30](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) When we view this state machine in the visualizer it looks very similar, except instead of having an entry action on this `pending` state, we see that it's invoking the service of `fetchData`. Then our done action is associated with the `fetchData` service and the error action is associated with the `fetchData` service.

[2:50](https://egghead.io/lessons/react-invoke-an-xstate-service-when-entering-a-state) When we fetch, we go into `pending`. The done action takes us to `successful` and the error action takes us to `fail`.