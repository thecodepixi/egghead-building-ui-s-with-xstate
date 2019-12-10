# 5: react-control-transitions-order-with-parallel-states

# **Transcript**

Instructor: [00:00](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=0) We're going to add some functionality to our app to be able to match a person with their homeworld. In order to do this, we'll create a `matchingMachine`. Let's outline the states first.

[00:12](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=12) There'll be an `answering` state and a `submitted` state. The `submitted` state will have its own substates of `correct` and `incorrect`. Let's add an `evaluating` state here with an automatic transition that sends to the `correct` state if the `isCorrect` `guard` return is `true`. Otherwise, it'll go to the `incorrect` state.

[00:46](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=46) We'll set the `initial` state of the `submitted` state to `evaluating` and the `initial` state of the overall machine to `answering`. Now, `answering` also has some substates. We need to keep track of the `selection` value of the `topList` and the `bottomList`.

**matching.ts** 

    import { Machine, assign } from 'xstate'; 
    
    export const matchingMachine = Machine({ 
    	id: 'matching',
    	initial: 'answering' 
    	states: {
    		answering: {
    			states: {
    				topList: {},
    				bottomList: {}
    			}
    		},
    		submitted: {
    			initial: 'evaluating' 
    			states: {
    				evaluating: { 
    					on: {
    						'': [
    							{ target: 'correct', cond: 'isCorrect' },
    							{ target: 'incorrect' }
    						]
    					}
    				},
    				correct: {},
    				incorrect: {}
    			}
    		},
    	}
    })

[01:08](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=68) Each of these lists can be either `unselected` or `selected`, but there's something different about `topList` and `bottomList` as compared to `unselected` and `selected`. **In this machine, you cannot be both `unselected` and `selected` at the same time. They're mutually exclusive.**

    ... 
    
    	states: {
    		answering: {
    			states: {
    				topList: {
    					states: {
    						unselected: {},
    						selected: {}
    					}
    				},
    				bottomList: {
    					states: {
    						unselected: {},
    						selected: {}
    					}
    				}
    			}
    		},
    
    ... 

[01:29](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=89) When we first enter the `answering` state, the `topList` is `unselected` and the `bottomList` is also `unselected`. We're essentially in both the `topList` and `bottomList` states at the same time. We call this situation a `parallel` state. We'll set our `initial` values for the list and let's see what this looks like in the visualizer.

    ... 
    
    	states: {
    		answering: {
    			type: 'parallel',
    			states: {
    				topList: {
    					initial: 'unselected',
    					states: {
    						unselected: {},
    						selected: {}
    					}
    				},
    				bottomList: {
    					initial: 'unselected',
    					states: {
    						unselected: {},
    						selected: {}
    					}
    				}
    			}
    		},
    
    ... 

[01:54](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=114) You can see that the `parallel` states have this dash border around them. We're currently in both the `topList unselected` state and the `bottomList unselected` state. Let's add some transitions for this machine.

[02:06](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=126) We need to set up some `context` here to keep track of the selected items. We'll track the top selected item and the bottom selected item. **When we're in the top `unselected` state, a `SELECT_TOP` event should transition us to `selected` and trigger a `setTopSelectedItem` action.** Same thing goes for the bottom list, only, we're selecting bottom (`SELECT_BOTTOM`) and `setBottomSelectedItem`.

    export const matchingMachine = Machine({ 
    	id: 'matching',
    	initial: 'answering', 
    	context: {
    		topSelectedItem: undefined, 
    		bottomSelectedItem: undefined
    	},
    	states: {
    		answering: {
    			type: 'parallel',
    			states: {
    				topList: {
    					initial: 'unselected',
    					states: {
    						unselected: {
    							on: {
    								SELECT_TOP: { target: 'selected', actions: 'setTopSelectedItem' }
    							}
    						},
    						selected: {
    							type: 'final' 
    						}
    					}
    				},
    				bottomList: {
    					initial: 'unselected',
    					states: {
    						unselected: {
    							on: {
    								SELECT_BOTTOM: { target: 'selected', actions: 'setBottomSelectedItem' }
    							}
    						},
    						selected: {
    							type: 'final' 
    						}
    					}
    				}
    			}
    		},
    		
    ...

[02:44](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=164) **XState also has the concept of a `final` state. In our case, we want to set these `selected` states to `final` and this lets us add an `onDone` transition to the `answering` state.** The `onDone` transition is fired as soon as all the substates are in their `final` state. Let's check this in the visualize one more time.

    export const matchingMachine = Machine({ 
    ...
    	states: {
    		answering: {
    			type: 'parallel',
    			onDone: 'submitted', 
    			states: {
    				topList: {
    					initial: 'unselected',
    					states: {
    						...
    						selected: {
    							type: 'final' 
    						}
    					}
    				},
    				bottomList: {
    					initial: 'unselected',
    					states: {
    						...
    						selected: {
    							type: 'final' 
    						}
    					}
    				}
    			}
    		},
    		
    ...

[03:08](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=188) Now we can select the top item and the bottom item and it will automatically transition to the `submitted` state. Also, if we select the bottom item first and then the top, it will also transition to the `submitted` state.

[03:22](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=202) Now, I'll create some types for this machine. We have the `states` (`MatchingSchema`) defined, the `events` and the `context`. 

    import { Machine, assign } from 'xstate';
    
    interface MatchingSchema {
    	states: {
    		answering: {
    			states: {
    				topList: {
    					states: {
    						unselected: {};
    						selected: {}; 
    					};
    				};
    				bottomList: {
    					states: {
    						unselected: {};
    						selected: {}; 
    					};
    				};
    			};
    		};
    		submitted: {
    			states: {
    				evaluating: {};
    				correct: {};
    				incorrect: {};
    			};
    		};
    	};
    }
    
    type MatchingEvents = 
    	| { type: 'SELECT_TOP'; selectedItem: any }
    	| { type: 'SELECT_BOTTOM'; selectedItem: any }
    	| { type: 'RESET' }
    
    // The context (extended state) of the machine
    interface MatchingContext {
    	topSelectedItem: any | undefined; 
    	bottomSelectedItem: any | undefined; 
    }
    
    export const matchingMachine = Machine<MatchingContext, MatchingSchema, MatchingEvents>({ 
    	... 
    }); 

[03:39](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=219) Let's define the `setTopSelectedItem` and `setBottomSelectedItem` actions. We'll set `topSelectedItem` to the e`vent.selectedItem` and the same thing for the `bottomSelectedItem`. Now let's use the state machine. We'll use the `useMachine` hook and we'll define the `isCorrect` guard, or `return true` if the `topSelectedItem` has a Homeworld that matches the `bottomSelectedItem.url`.

    export const matchingMachine = Machine<MatchingContext, MatchingSchema, MatchingEvents>({ 
    	...
    
    	actions: {
    		setTopSelectedItem: assign((ctx, event: any) => ({ 
    			topSelectedItem: event.selectedItem
    		})),
    		setBottomSelectedItem: assign((ctx, event: any) => ({ 
    			bottomSelectedItem: event.selectedItem
    		})),
    	}	
    }); 

**App.tsx**

    ... 
    import { matchingMachine } from './machines/matching'
    
    function App() {
    	const [matchingState, sendToMatchingMachine] = useMachine(matchingMachine, {
    		guards: {
    			isCorrect: (ctx) => {
    				return ctx.topSelectedItem.homeworld === ctx.bottomSelectedItem.url;
    			}
    		}
    	})	
    	... 
    }

[04:10](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=250) I've updated the UI here. I've added a `button` **inside of each list item that will send a select event to the `matchingMachine` when it's clicked** and have a `backgroundColor` to show that it's been selected.

    ... 
    return ( 
    	<div classname="App"> 
    	 ... 
    		<ul> 
    			... 
    			<li key={index}> 
    				<button style=
    					{{backgroundColor: matchingState.context.topSelectedItem === person ? 'lightblue' : ''}}
    					onClick={ () => 
    						sendToMatchingMachine({ 
    							type: 'SELECT_TOP',
    							selectedItem: person 
    						}) 
    					}
    				> 
    					{person.name}
    				</button>
    )

[04:24](https://egghead.io/lessons/react-control-transitions-order-with-parallel-states#t=264) I also display a message for the `correct` answer and for an `incorrect` answer. Let's see what this looks like. Now, I can select the top item and then the bottom item. Or I can select the bottom item and then the top item.

    		...
    
    		{matching.State.matches('submitted.correct') ? (
    			<p>The Force is strong with this one.</p>
    		) : null} 
    		{matching.State.matches('submitted.incorrect') ? (
    			<p>Do or do not. There is no try.</p>
    		) : null} 
    
    		...