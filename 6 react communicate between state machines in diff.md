# 6: react-communicate-between-state-machines-in-different-components

Instructor: [00:00](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=0) This app component is using two different machines, the `matchingMachine` and the `fetchMachine`. Also, **the `fetchMachine` is used twice, once for people and once for planets. Let's create a list component that will use the fetch machine and handle rendering a list**.

[00:15](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=15) **We'll just copy everything in the `App` component,** make a new `List.tsx` file, paste in the `App` component content and then delete what we don't need. Change our references to `app` to `list`. `fetchPeople` and `fetchPlanets` will be passed to us from the **parent component** and **remove all references to the matching machine.**

[00:36](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=36) **We'll rename `fetchPeople` state to `fetchDataState`** and `sendToPeopleMachine` should become `sendToDataMachine`. **Let's add a prop here for `fetchData` and use that here**. We don't need the planet state, we can remove that. Scroll down. W**e'll come back to fix the matching state references.**

**List.tsx** 

    import { useMachine } from '@xstate/react';
    import React, { useEffect } from 'react';
    import './List.css';
    import { fetchMachine } from './machines/fetch';
    
    function List({ fetchData }) { 
    
    	const [fetchDataState, sendToDataMachine] = useMachine(fetchMachine, { 
    		services: {
    			fetchData: () => fetchData().then(r => r.results)
    		}
    	});
    	userEffect(() => { 
    		sendToDataMachine({ type: 'FETCH' });
    	}, [sendToDataMachine]); 
    }
    
    ... 

[01:05](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=65) **Now we just need to fix these references to the matching machine.** We'll have the parent component **pass in the selected item** and we'll use an `onSelection` prop **pass back the item that was selected**. Here let's rename `person` to `item`.

    ... 
    return ( 
    	<div classname="App"> 
    	 ... 
    		<ul> 
    			... 
    			<li key={index}> 
    				<button style=
    					{{backgroundColor: selectedItem === item ? 'lightblue' : ''}}
    					onClick={ () => 
    						onSelection(item)
    					}
    				> 
    					{person.name}
    				</button>
    ...
    
    )

[01:21](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=81) **Now we needed to find these props: `selectedItem`, `onSelection`.** `fetchData` is a function that returns a promise with results that's an array. `selectedItem` could be anything. `onSelection` takes any `item` and `returns void`.

    ... 
    
    function List({ fetchData, selectedItem, onSelection }: {
    	fetchData: () => Promise<{results: any[]}>;
    	selectedItem: any;
    	onSelection: (item: any) => void;
    }) { 
    	...
     }

[01:50](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=110) Let's clean up the styles on this button to use a `class` instead. If it's `selected`, it should have the `selected` class **and it should always have a class of `list-button`**. Let's update the app component. Don't need the fetch machine anymore. We'll use the list component here and import it.

    ... 
    return ( 
    			... 
    			<li key={index}> 
    				<button 
    					className={ 'list-button ' + (selectedItem === item ? 'selected' : '')}
    					style=
    					{{backgroundColor: selectedItem === item ? 'lightblue' : ''}}
    					onClick={ () => 
    						onSelection(item)
    					}
    				> 
    					{person.name}
    				</button>
    ...
    
    )

**App.tsx**

    import { useMachine } from '@xstate/react';
    import React, { useEffect } from 'react';
    import { fetchPeople, fetchPlanets } from './api';
    import { List } from './List';
    import './App.css';
    import { matchingMachine } from './machines/matching'; 
    
    ... 
    
    return ( 
    			...
    		<> 
    		<List> </List>
    		<hr> 
    	...
     
    ) 
    

[02:21](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=141) `fetchData` for this one is `fetchPeople`. Selected item is `matchingState.context.topSelectedItem`. `onSelection`, we'll take the `selectedItem` and send to the match machine. Select top event with the selected item.

    ... 
    
    return ( 
    			...
    		<> 
    		<List 
    			fetchData={fetchPeople} 
    			selectedItem={matchingState.context.topSelectedItem} 
    			onSelection={(selectedItem) => { 
    				sendToMatchingMachine({ type: 'SELECT_TOP', selectedItem }); 
    			}}
    		></List>
    		<hr/> 
    	...
     

[02:48](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=168) Do the same thing for planets. `fetchPlanets`, `bottomSelectedItem`, and `selectBottom`. Now, let's see if that worked. Just like before, we can select the top and the bottom or the bottom and then the top.

    ... 
    	<List 
    			fetchData={fetchPlanets} 
    			selectedItem={matchingState.context.topSelectedItem} 
    			onSelection={(selectedItem) => { 
    				sendToMatchingMachine({ type: 'SELECT_TOP', selectedItem }); 
    			}}
    		></List>
    	</> 

[03:09](https://egghead.io/lessons/react-communicate-between-state-machines-in-different-components#t=189) With this refactor, it's clear that the `matchingMachine` knows nothing about the `fetchMachine`. They simply coordinate their actions by sending messages through components props.