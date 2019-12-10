# 7: react-save-previous-states-with-a-history-state

# **Transcript**

Instructor: [00:00](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=0) I've modified the matching machine to wrap the answering state in a `quiz` state. This allowed me to add a verifying step where you can verify your answers before you choose to submit them or go back and change them again.

[00:14](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=14) I've also removed the `onDone` transition and replaced it with a manual `continue` event, **so the user can decide when they want to move on.** **You can continue and then change answers will take you back to the `answering` state.** I still need to write a transition to go from the `verifying` state over to the `submitted` state.

[00:33](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=33) **Let's add that in the code here.** We'll add a `submit` event. **Because submit it is not a sibling to the `verifying` state, we need to reference it by ID and add that ID to the submitted state**. If we update, the `verifying` state has a transition directly to the `submitted` state.

**[xstate.js.org/vi](http://xstate.js.org/vix/)z/ (visualization of `fetch.ts`)** 

    ... 
    
    verifying: {
    	on: {
    		CHANGE_ANSWERS: 'answering',
    		SUBMIT: '#submitted'
    	} 
    },
    submitted: { 
    	id: 'submitted', 
    	... 
    }
    
    ... 

[00:53](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=53) **There's an issue with this machine, though.** Say, we select some items and then continue to the `verifying` state, and realize we want to change them. When we come back to the `answering` state, we've returned in the initial `unselected` state for each of the list. **We need some way for the state machine to remember what state it was in. This is where we can use history states.**

[01:14](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=74) We'll add a `hist` state as a sibling to the `topList` and `bottomList` and give it a `type` of `history`. **What a history state does, is it keeps track of any transitions that happen to any of its sibling states. Then, when the state machine transitions to that history state, the history state redirects to the most recently entered sibling state.**

[01:38](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=98) **We can specify that the `history` state is a `history` state of type `shallow`, which is the default, or `deep`, which will remember any transitions to siblings or their sub-states.** Now we just update this `CHANGE_ANSWERS` transition so that it goes to `answering.hist`, instead of the higher level `answering` state.

    ... 
    
    		hist: {
    			type: 'history',
    			history: 'deep' 
    		}
    	}
    }, 
    verifying: {
    	on: {
    		CHANGE_ANSWERS: 'answering.hist',
    		SUBMIT: '#submitted'
    	} 
    }
    	
    ... 

[01:59](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=119) Now if we select the top list, but leave the bottom list unselected, and then continue to `verifying` and return back to the `history` state, the top list is selected and the bottom list is unselected.

[02:13](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=133) Let's add the UI for this `verifying` state. We'll add a `button` to `CHANGE_ANSWERS` and one to `SUBMIT`. I need to reference the `verifying` state by its parent, saying, `quiz.verifying`. We'll add a `button` to the `answering` state to `CONTINUE`. Let's see what that looks like.

**App.tsx**

    return ( 
    	<div className='App'> 
    	{matchingState.matches('quiz.answering') ? ( 
    		<>
    			<button onClick={() => sendToMatchingMachine({ type: 'CONTINUE' })}>
    				Continue
    			</button> 
    			<List 
    				...
    			></List>
    		<hr/> 
    		... 
    	}}
    ... 
    {matchingState.matches('quiz.verifying') ? (
    	<>
    	<p>
    	... 
    	</p>
    	<button onClick={() => sendToMatchingMachine({ type: 'CHANGE_ANSWERS' })} >
    		Change Answers 
    	</button> 
    	<button onClick={() => sendToMatchingMachine({ type: 'SUBMIT' })} >
    		Submit
    	</button> 
    	</>
    ... 
    )

[02:39](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=159) We can select R2D2 and Yavin4 and choose continue. Then, if we're not sure, we can go back to change answers and that remembers that we're in a selected state. Then, we can continue and when we are ready, summit our answer.

[02:58](https://egghead.io/lessons/react-save-previous-states-with-a-history-state#t=178) **One thing to watch out for with `history` states is that you almost never want to transition from a sibling state to a history state, because all that would do is transition immediately back to the state you came from.** Any transitions into this history state should come from outside of this answering state.