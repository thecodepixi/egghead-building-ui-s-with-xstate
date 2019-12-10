# 8: react-execute-code-when-entering-and-exiting-states-using-activities

Instructor: [0:00](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) I've created a small state machine to model the two sides of the force. When you're on the light side of the force and you're corrupted, you go to the dark side of the force. On the dark side of the force, if you're redeemed, you go back to the light side of the force.

[0:14](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) We'll use this state machine to explain the concept called **activities**. **Activities are useful when there's a process that needs to start when you enter a state and end when you leave the state.**

**[xstate.js.org/viz/](http://xstate.js.org/viz/) (visualization of `force.ts`)**

    const forceMachine = Machine( 
    	{
    		id: 'force',
    		initial: 'light',
    		context: {},
    		states: {
    			light: {
    				on: {
    					CORRUPT: 'dark'
    				}
    			},
    			dark: {
    				activities: ['theDarknessGrows'] 
    				on: {
    					REDEEM: 'light'
    				}
    			}
    		}
    	},
    	{}
    );

[0:25](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) This is a great place for something like an observable that needs to be subscribed and then unsubscribed, or a set interval that needs to be canceled or WebSocket that needs to be connected and then disconnected. We'll add an activity called `theDarknessGrows` to the dark side.

[0:44](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) Let's use this machine in our `App` component. We'll define this activity in our second object here, `theDarknessGrows`. **An activity is a function that takes in the context and returns another function that will be executed when the state machine leaves the dark state.**

[1:08](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) Code up here is when you're entering the dark state, and down here is when you're leaving the dark state. **We'll create a local component state variable called `darkSidePower`. We'll set it to zero initially.**

**App.tsx** 

    ... 
    
    function App() { 
    	... 
    	
    	const [darkSidePower, setDarkSidePower] = useState<number>(0); 
    	const [forceState, sendToForceMachine] = useMachine(forceMachine, { 
    		activities: {
    			theDarknessGrows: (ctx) => {
    				// entering dark state
    				return () => { 
    				// leaving dark state 
    				} 
    			}
    		}
    	}) 
     

[1:22](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) When we enter the dark side, we'll set the `darkSidePower` to `10`. Then, we'll set up an `interval` here to periodically increase that power. We'll do that every `600` milliseconds. Now, when we leave the dark state, we'll set the `darkSidePower` back to and `clear` that `interval`.

    ... 
    
    theDarknessGrows: (ctx) => {
    	// entering dark state
    	setDarkSidePower(10);
    	const interval = setInterval( 
    		() => setDarkSidePower(power => power + 10), 600 
    	); 			
    	return () => { 
    	// leaving dark state 
    		setDarkSidePower(0); 
    		clearInterval(interval); 
    	};
    
    ...

[1:42](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) Let's make a `button` that allows us to change sides of the force. If the force state matches `light`, we'll make a button that says, "Come to the dark side," and send to the force machine a `CORRUPT` event. Otherwise, we'll say, "Go back to the light side," and we'll send a `REDEEM` event.

[2:11](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) Let's use the `darkSidePower` by sending it as a prop into a `list` component. Say `darkSidePower = darkSidePower`. Let's set that up in the list component.

    ... 
    
    return ( 
    	<div className='App'> 
    	{matchingState.matches('quiz.answering') ? ( 
    		...
    			</button> 
    
    			{forceState.matches('light') ? ( 
    			<button onClick={() => sendToForceMachine({ type: 'CORRUPT' })}>
    				Come to the Dark Side 
    			</button> 
    			: 
    			<button onClick={() => sendToForceMachine({ type: 'REDEEM' })}>
    				Go Back to the Light Side
    			</button> 
    			} 
    		<> 
    		<List 
    			fetchData={fetchPeople} 
    			selectedItem={matchingState.context.topSelectedItem} 
    			onSelection={(selectedItem) => { 
    				sendToMatchingMachine({ type: 'SELECT_TOP', selectedItem }); 
    			darkSidePower={darkSidePower}
    			}}
    		></List>
    		<hr/> 
    	}}
    
    	... 
    )

[2:27](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) `darkSidePower` here, and we'll define it as a number. We'll add some styles to the list item. Let's set the `position` to `relative` , `top` to a `randomValue` based on the `darkSidePower`, `left` `randomValue` also based on the darkSidePower. Let's add some `rotation` as well and make the `transition` to these values.

**List.tsx**

    ... 
    export function List ({ 
    	...
    	darkSidePower
    }, { 
    	...
    	darkSidePower: number; 
    }) 
    
    ... 
    
    <li key={index} 
    	style={{ 
    		position: 'relative',
    		top: randomValue(darkSidePower),
    		left: randomValue(darkSidePower), 
    		transform: `rotate(${randomValue(darkSidePower)}deg)`,
    		transition: 'all 1s ease' 
    	}}
    

[3:06](https://egghead.io/lessons/react-execute-code-when-entering-and-exiting-states-using-activities) If we click "Come to the dark side," our answer starts to get more difficult to select. It only gets worse as the power of the dark side gets stronger and stronger until finally, if we can make it back to the light side, they return back to the normal place.