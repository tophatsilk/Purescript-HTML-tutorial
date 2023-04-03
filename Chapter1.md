[Introduction](./Introduction.md) | [Chapter 1. The basics.](./Chapter1.md) | [Chapter 2. HTML properties.](./Chapter2.md) | [Chapter 3. Events.](./Chapter3.md) | [Chapter 4. Basic data storage.](./Chapter4.md) | [Chapter 5. Canvas and images.](./Chapter5.md)
# Purescript web programming basics tutorial: Halogen versus purescript HTML. 
# Chapter 1 - the basics

In this chapter we will show how to build the essential, basic parts of an HTML page using basic purescript HTML. We will provide all the necessary tools to recreate the first example in the Halogen Guide, without using Halogen itself.
## First steps: Adding an element to the DOM
To become familiar with purescript, we will begin with a basic example. The code may seem to be very long, but most of it are comments to improve understanding. This code is basically all you need to build a static HTML page with divisions.

You can copy the following code in Fig. 1.1, and try it in: https://try.purescript.org by pasting it in the left code-column.


The code defines a module Main (first line) and imports several standard modules we need. After that we define three of our own functions (selectDivTargetFromDocument, updateText, and maybeAppendChildNode) which are just to improve the readability of our main code.
For now we will scroll downward and take a look at the main function.
```
module Main where

import Prelude

import Data.Maybe (Maybe(..))
import Effect (Effect)
import Web.DOM.Document (createElement)
import Web.DOM.Element (Element, toNode, setId)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
import Web.HTML (window)
import Web.HTML.HTMLDocument as HTMLDoc
import Web.HTML.Window (document)

-- Take the document ('doc') itself and select from this document an element
-- (division) by its id. This division will be the target element for any 
-- changes.
-- Parameters: 
-- doc - the (HTML) document to be manipulated
-- divId - a String representing the HTML id of the division of element.
selectDivTargetFromDocument :: String -> HTMLDoc.HTMLDocument -> Effect (Maybe Element)
selectDivTargetFromDocument divId doc = querySelector (QuerySelector (divId)) (HTMLDoc.toParentNode doc)

-- Take a string and replace the content of an element with this string. If there is no element, do nothing.
-- Parameters:
-- str - the string representing the new content of the element
-- el - the element to be altered
updateText :: String -> Maybe Element -> Effect Unit
updateText str (Just el) = setTextContent str (toNode el)
updateText _ _ = pure unit

-- Append a child if a parent element is given.
-- The function appendChild from Web.DOM.Node takes nodes as arguments 
-- (appendChild :: Node → Node → Effect Unit), so the elements need to be
-- converted to nodes with toNode from Web.DOM.Element.
maybeAppendChildNode ∷ Maybe Element → Element → Effect Unit
maybeAppendChildNode (Just parent1) child1 = appendChild (toNode child1) (toNode parent1)
maybeAppendChildNode _ _ = pure unit

main :: Effect Unit
main = do
  -- Getting the basic 'materials'.
  -- Obtain the document window.
  w <- window
  -- Extract the HTMLdocument from it using document from Web.HTML.Window
  d <- document w
  -- We need to select the document 'body' as a parent for the element to be created.
  bodyEl <- selectDivTargetFromDocument "body" d
  -- The above code to get the 'body' element may be rewritten using 'bind' (=<<) to shorten it to:
  -- bodyEl <- selectDivTargetFromDocument "body" =<< document =<< window

  -- The createElement (createElement :: String → Document → Effect Element) function  from
  -- Web.DOM.Document uses a (non HTML-) document to create an element, so we have to create
  -- that first from the document. (<$> is an alias for Data.Functor.map. It is not relevant
  -- to this example, just a shortcut.)
  nonHTMLdoc <- HTMLDoc.toDocument <$> (document =<< window)

  -- Start the work.
  -- Now we may create an element of type "div" (HTML-tag: <div>).
  createdElement <- createElement "div" nonHTMLdoc
  -- And add the created element to the body.
  maybeAppendChildNode bodyEl createdElement
  -- If we want to, we may set the HTML id of the element. 
  -- (We do not need a return value, so we use an anonymous function.)
  _ <- setId "testDiv" createdElement

  -- Now we may alter the text content of the created element.
  -- (updateText takes a Maybe element as argument, so we use
  -- "Just createdElement" with Just as a constructor).
  updateText "test" (Just createdElement)
  
  -- The last statement in a 'do' block must be an expression,
  -- so we add the simplest expression.
  pure unit
```
**Figure 1.1: Listing of code to add DOM element.**

The above code in Fig. 1.1 contains the steps you need to build a basic web page. 

First we obtain the relevant elements of the HTML document to work with; getting the basic materials. This is hardly any different from standard javascript. Purescript is just more strict in its types and that is what you see: The elements need to be converted to the proper types.
After that we can start building:
- We create a new element of type "div".
- Append it to the body of the document.
- (Optionally) Set the HTML id of the element.
- Put some text in it, if we want to.

In the above example in Fig. 1.1 we took all the necessary steps to build up an HTML element, but we can streamline the above code to contain one single function that takes the elements HTML tag type (e.g. 'div' of 'button') and its text. At the moment we do not need to add an HTML id, so we leave that out as well. This gives the following, much shorter code:
```
module Main where

import Prelude

import Data.Maybe (Maybe(..))
import Effect (Effect)
import Web.DOM.Document (Document, createElement)
import Web.DOM.Element (Element, setId, toNode)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
import Web.HTML (window)
import Web.HTML.HTMLDocument as HTMLDoc
import Web.HTML.Window (document)

-- Take the document ('doc') itself and select from this document an element (division) by its id.
-- This division will be the target element for any changes.
-- Parameters: 
-- divId - a String representing the HTML id of the division of element.
-- doc - the (HTML) document to be manipulated
selectDivTargetFromDocument :: String -> HTMLDoc.HTMLDocument -> Effect (Maybe Element)
selectDivTargetFromDocument divId doc = querySelector (QuerySelector (divId)) (HTMLDoc.toParentNode doc)

-- Create an element with an HTML tag type and contents.
-- Parameters:
-- tag - the HTML tag type, e.g. 'div' or 'button'.
-- contents - the text content of the element, as string.
-- nonHTMLdoc - the (nonHTML)document itself.
createElementWithTagAndContent ∷ String → String → Document → Effect Element
createElementWithTagAndContent tag contents nonHTMLdoc = do
  -- Create the new element
  createdElement <- createElement tag nonHTMLdoc
  -- Add its contents. (No return needed, hence the anonymous function '_'.)
  _ <- setTextContent contents (toNode createdElement)
  -- Return the element.
  pure createdElement

-- Take a string and replace the content of an element with this string. If there is no element, do nothing.
-- Parameters:
-- str - the string representing the new content of the element
-- el - the element to be altered
updateText :: String -> Maybe Element -> Effect Unit
updateText str (Just el) = setTextContent str (toNode el)
updateText _ _ = pure unit

-- Append a child if a parent element is given.
-- The function appendChild from Web.DOM.Node takes nodes as arguments 
-- (appendChild :: Node → Node → Effect Unit), so the elements need to be
-- converted to nodes with toNode from Web.DOM.Element.
maybeAppendChildNode ∷ Maybe Element → Element → Effect Unit
maybeAppendChildNode (Just parent1) child1 = appendChild (toNode child1) (toNode parent1)
maybeAppendChildNode _ _ = pure unit


main :: Effect Unit
main = do  
  -- Get the body element and the (nonHTML)document
  bodyEl <- selectDivTargetFromDocument "body" =<< document =<< window
  nonHTMLdoc <- HTMLDoc.toDocument <$> (document =<< window)

  -- Then we create an element of type "div" (HTML-tag: <div>).
  createdElement <- createElementWithTagAndContent "div" "new div" nonHTMLdoc
  -- And append it to the body.
  maybeAppendChildNode bodyEl createdElement
```
**Figure 1.2: Listing of shorter code to add DOM element.**

To show how short the code really became, we will show the code without comments or type declarations. **We strongly advise against using code without commments!** It may seem pretty now, but it is annoying for people who want or need to read it later, and it may even annoy yourself when you read your own code after a year or so.
```
module Main where

import Prelude
import Data.Maybe (Maybe(..))
import Effect (Effect)
import Web.DOM.Document (Document, createElement)
import Web.DOM.Element (Element, setId, toNode)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
import Web.HTML (window)
import Web.HTML.HTMLDocument as HTMLDoc
import Web.HTML.Window (document)

selectDivTargetFromDocument divId doc = querySelector (QuerySelector (divId)) (HTMLDoc.toParentNode doc)

createElementWithTagAndContent tag contents nonHTMLdoc = do
  createdElement <- createElement tag nonHTMLdoc
  _ <- setTextContent contents (toNode createdElement)
  pure createdElement

maybeAppendChildNode (Just parent1) child1 = appendChild (toNode child1) (toNode parent1)
maybeAppendChildNode _ _ = pure unit


main :: Effect Unit
main = do  
  bodyEl <- selectDivTargetFromDocument "body" =<< document =<< window
  nonHTMLdoc <- HTMLDoc.toDocument <$> (document =<< window)
  createdElement <- createElementWithTagAndContent "div" "new div" nonHTMLdoc
  maybeAppendChildNode bodyEl createdElement
```
**Figure 1.3: Listing of shorter code of Fig. 1.2, without comments or type declarations. <p style="color:Red;">We strongly recommend using the code in Fig. 1.2!</p>**

So, we created the first bit of purescript to make a web page with a single element with some text.

But this is all 'basic HTML' as the Halogen Guide refers to. So, how about we add some 'response to DOM events'. Something beyond the scope of "pure functions that produce HTML", mentioned in the Halogen Guide.

## Step 2: Responding to DOM events
In the above code in Figs. 1-3 we altered the DOM in several ways: we added an element, but we also changed the content of that element. 

### Buttons
Now if we would want to add a button, instead of a division element, the only change needed would be to alter the 'createdElement' assignment in Fig. 1.1 as follows:
```
createdElement <- createElement "button" nonHTMLdoc
```
Or change the tag in createElementWithTagAndContent in main in Figs. 2-3:
```
createdElement <- createElementWithTagAndContent "button" "button text" nonHTMLdoc
```
You can try this in try.purescript.org. You will immediately get a button instead of a normal text division. But clicking it does nothing ...

### ... and clicks
Now, if you want to obtain a response to this button, there are several ways and several modules that you can use. But we will use the most elementary approach by adding an eventListener to the button. 

We will use the code from Fig. 1.2 above, and add some functions. Again, the code listing may seem daunting, but we will show we only added some logical elements. The parts between the lines are the same as before.
```
module Main where

import Prelude

import Data.Maybe (Maybe(..))
import Effect (Effect)
import Web.DOM.Document (Document, createElement)
import Web.DOM.Element (Element, setId, toNode, toEventTarget)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
import Web.Event.Event (Event, EventType(EventType))
import Web.Event.EventTarget (EventListener, addEventListener, eventListener)
import Web.HTML (window)
import Web.HTML.HTMLDocument as HTMLDoc
import Web.HTML.Window (document)

---------------------------------------------------------------------------
-- The part below is the same as before

-- Take the document ('doc') itself and select from this document an element (division) by its id.
-- This division will be the target element for any changes.
-- Parameters: 
-- doc - the (HTML) document to be manipulated
-- divId - a String representing the HTML id of the division of element.
selectDivTargetFromDocument :: String -> HTMLDoc.HTMLDocument -> Effect (Maybe Element)
selectDivTargetFromDocument divId doc = querySelector (QuerySelector (divId)) (HTMLDoc.toParentNode doc)

-- Create an element with an HTML tag type and contents.
-- Parameters:
-- tag - the HTML tag type, e.g. 'div' or 'button'.
-- contents - the text content of the element as string.
-- nonHTMLdoc - the (nonHTML)document itself.
createElementWithTagAndContent ∷ String → String → Document → Effect Element
createElementWithTagAndContent tag contents nonHTMLdoc = do
  -- Create the new element
  createdElement <- createElement tag nonHTMLdoc
  -- Add its contents (no return needed, hence the anonymous function '_'.)
  _ <- setTextContent contents (toNode createdElement)
  -- Return the element.
  pure createdElement

-- Append a child if a parent element is given.
-- The function appendChild from Web.DOM.Node takes nodes as arguments 
-- (appendChild :: Node → Node → Effect Unit), so the elements need to be
-- converted to nodes with toNode from Web.DOM.Element.
maybeAppendChildNode ∷ Maybe Element → Element → Effect Unit
maybeAppendChildNode (Just parent1) child1 = appendChild (toNode child1) (toNode parent1)
maybeAppendChildNode _ _ = pure unit

---------------------------------------------------------------------------
-- *** The code above is the same as before, except for some imports **
-- We define three new functions, and recall our earlier updateText function:

-- Define the EventType. In this case: "click". (This could also be: "input" or "mouseover".)
inputEvent :: EventType
inputEvent = EventType ("click")

-- Add an eventListener, if an element is provided.
-- Parameters:
-- eh - the EventListener to be used.
-- el - the element to be equipped with the listener.
maybeAddEventListener ∷ EventListener → Maybe Element → Effect Unit
maybeAddEventListener eh (Just el) = addEventListener inputEvent eh true (toEventTarget el)
maybeAddEventListener _ _ = pure unit

-- Perform this action when the event is triggered:
-- Put "You clicked" content in the output division (we do not use the event 'evt' itself, for now).
inputEventHandler :: Maybe Element -> Event -> Effect Unit
inputEventHandler el evt = do
  updateText "You clicked" el
  
-- Take a string and replace the content of an element with this string. If there is no element, do nothing.
-- Parameters:
-- str - the string representing the new content of the element
-- el - the element to be altered
updateText :: String -> Maybe Element -> Effect Unit
updateText str (Just el) = setTextContent str (toNode el)
updateText _ _ = pure unit

main :: Effect Unit
main = do
---------------------------------------------------------------------------
-- Again, we use the same code to start:

   -- Get the body element and the (nonHTML)document
  bodyEl <- selectDivTargetFromDocument "body" =<< document =<< window
  nonHTMLdoc <- HTMLDoc.toDocument <$> (document =<< window)

---------------------------------------------------------------------------
-- *** The code for main above is the same as before. **

-- First we may create an element of type "div" (HTML-tag: <div>).
  divElement <- createElementWithTagAndContent "div" "text" nonHTMLdoc
  -- And add the created element to the body.
  maybeAppendChildNode bodyEl divElement

  -- Now we may create an element of type "button" (HTML-tag: <button>).
  buttonElement <- createElementWithTagAndContent "button" "button" nonHTMLdoc
  -- And add the created element to the body.
  maybeAppendChildNode bodyEl buttonElement
  
  -- We create an eventListener that activates the inputEventHandler function. 
  -- In this case the inputEventHandles takes an outputElement (Just divElement) as input, 
  -- because that is the target for the effects of the click.
  eh <- eventListener (inputEventHandler (Just divElement))

  -- And finally add it to the chosen element (the button element).
  maybeAddEventListener eh (Just buttonElement)
  
```
**Figure 1.4: Listing of simple code for a button with eventListener.**

Again, you can try this code in https://try.purescript.org. You will see a button and a division above the button of which the content changes when you click the button.

So, what did we do?

### What did we do to click?
 First of all, the first part of the code, up to the line, is the same as in the listing in Fig. 1.1, except for some imports from the Web.Event modules. Then we added some new functions. 

The first one is very interesting: we added an EventType (https://pursuit.purescript.org/packages/purescript-web-events/4.0.0/docs/Web.Event.Event#t:EventType). In this case we used 'click', which refers to the mouse click, but the EventType could also be 'input' referring to the user typing some text, or 'mouseover' which would mean the event would react to the mouse moving over the element. So, this EventType gives us a lot of flexibility. 

**Note** Pursuit does not give you a lot of help on which EventTypes are available. This, I personally feel, is typical for purescript in general: the documentation lacks examples and deeper information. If you are confronted with this kind of problem, we suggest you look at javascript for the answers, and hope for the best...

We also made a function 'maybeAddEventListener'. This refers to the 'Maybe' monad, because there is no guarantee that the element we want to add a listener to, really exists at the time of adding the EventListener. If there is no element, the function does nothing. 

The addEventListener function we used in our function is from the Web.Event.EventTarget module and has the following type:
```
addEventListener :: EventType -> EventListener -> Boolean -> EventTarget -> Effect Unit
```
The EventType is the one we saw before (inputEvent). The EventListener will created in the main function, as well as the EventTarget. The boolean argument indicates whether the listener should be added for the "capture" phase. This is not relevant for the current paper, but if you wish to understand it, we refer to: https://javascript.info/bubbling-and-capturing.

And, finally, we added an inputEventHandler in which we determine what will happen, if the event is triggered. In this function definition there is a referral to the 'evt' Event. We do not need it in this case, so we could have left the space open, like so:
```
inputEventHandler el _ = do
  updateText "You clicked" el
```
The above functions will help us in the main function.
### The main function for responding to clicks
In the main function we start out as before: 
- We create a division. Later on we will use this division as output Element.
- Then we make another element, but we give it the type "button", which will be the input element. 

    [Note: Up to here, we just made some elements we may use.]
- Now, we make an eventListener pointing to the eventHandler (which itself points to the output element, but that is just because we want to use that element to show something happened.)
- And finally, we put it all together by using our 'maybeAddEventListener' function with the just created eventListener ('eh') and our input element (the button).

### Result
Well, we created elements for the HTML DOM, and we can respond to actions upon them!

So, apparently, we can respond to DOM events, even without Halogen.
But, you may complain: "Is this simple HTML coding?!"

Basically: "Yes. Wait till we start to compare with Halogen..."
But, do not despair, because all the above code shows a lot of repetition, and once we start to create handy functions, creating HTML will not be a problem.

### Are we ready to compare with Halogen now? 
We could, but the Halogen Guide also refers to a "state that represents values over time", which would not be possible with "pure functions that produce HTML". So, let's see what we can do.

## Step 3: Using states to keep values over time.
We will use the  ```Ref``` type (https://pursuit.purescript.org/packages/purescript-refs/6.0.0/docs/Effect.Ref#v:new) for mutable value references. It is described in the Purescript by Example book in chapter 12 on canvas graphics (https://book.purescript.org/chapter12.html) in the paragraph "Global Mutable State".

For an interesting alternative tutorial on HTML programming with purescript, without Halogen, using a similar approach, we refer to Gabriel Crispino's excellent 'Moving box' example (https://levelup.gitconnected.com/building-a-moving-box-with-purescript-ae1a490429ab). But the example below is easier, so you might want to start with that one.

We will give a code example that yields the same HTML page as the first example ('tiny Halogen app') in the Halogen Guide. 

**Note:** Halogen creates a separate division for each component. So far, we added our elements to the body in Figs. 1-4. To obtain the exact same HTML as in the Halogen Guide we will first create a new division, add it to the body and then add the further divisions to this new 'main' division

```
module Main where

import Prelude

import Data.Maybe (Maybe(..))
import Effect (Effect)
import Effect.Ref (Ref, new, read, write)
import Web.DOM.Document (Document, createElement)
import Web.DOM.Element (Element, setId, toNode, toEventTarget)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
import Web.Event.Event (Event, EventType(EventType))
import Web.Event.EventTarget (EventListener, addEventListener, eventListener)
import Web.HTML (window)
import Web.HTML.HTMLDocument as HTMLDoc
import Web.HTML.Window (document)

---------------------------------------------------------------------------
-- The part below is the same as before

-- Take the document ('doc') itself and select from this document an element (division) by its id.
-- This division will be the target element for any changes.
-- Parameters: 
-- doc - the (HTML) document to be manipulated
-- divId - a String representing the HTML id of the division of element.
selectDivTargetFromDocument :: String -> HTMLDoc.HTMLDocument -> Effect (Maybe Element)
selectDivTargetFromDocument divId doc = querySelector (QuerySelector (divId)) (HTMLDoc.toParentNode doc)

-- Create an element with an HTML tag type and contents.
-- Parameters:
-- tag - the HTML tag type, e.g. 'div' or 'button'.
-- contents - the text content of the element as string.
-- nonHTMLdoc - the (nonHTML)document itself.
createElementWithTagAndContent ∷ String → String → Document → Effect Element
createElementWithTagAndContent tag contents nonHTMLdoc = do
  -- Create the new element
  createdElement <- createElement tag nonHTMLdoc
  -- Add its contents (no return needed, hence the anonymous function '_'.)
  _ <- setTextContent contents (toNode createdElement)
  -- Return the element.
  pure createdElement

-- Append a child if a parent element is given.
-- The function appendChild from Web.DOM.Node takes nodes as arguments 
-- (appendChild :: Node → Node → Effect Unit), so the elements need to be
-- converted to nodes with toNode from Web.DOM.Element.
maybeAppendChildNode ∷ Maybe Element → Element → Effect Unit
maybeAppendChildNode (Just parent1) child1 = appendChild (toNode child1) (toNode parent1)
maybeAppendChildNode _ _ = pure unit

---------------------------------------------------------------------------
-- *** The code above is the same as before, except for some imports **
-- Next, we use the same inputEvent, maybeAddEventListener, and updateText functions as before.
-- But we changed the inputEventHandler function by adding a stateChanger (function) parameter
-- and a stateRef that contains the state reference. And we added two simple functions (incrementer
-- and decrementer) we can use as stateChanger functions.

-- Define the EventType. In this case: "click". This may also be: "input" or "mouseover".
inputEvent :: EventType
inputEvent = EventType ("click")

-- Add an eventListener, if an element is provided.
-- Parameters:
-- eh - the EventListener to be used.
-- el - the element to be equipped with the listener.
maybeAddEventListener ∷ EventListener → Maybe Element → Effect Unit
maybeAddEventListener eh (Just el) = addEventListener inputEvent eh true (toEventTarget el)
maybeAddEventListener _ _ = pure unit

-- Perform this action when the event is triggered:
-- Parameters:
-- stateChanger - the function that will be used to alter the state upon event.
-- el -           the element to which the new text will be sent.
-- stateRef -     the state reference that will be altered
-- evt -          the event (again, not used in this function).
inputEventHandler ∷ (Int -> Int) -> Maybe Element → Ref { number ∷ Int } → Event → Effect Unit
inputEventHandler stateChanger el stateRef evt = do
  -- Get the state
  state <- read stateRef
  -- Get a new number for the number in the state, by using the stateChanger function (like the 
  -- function incrementer below).
  let newNumber = stateChanger state.number
  -- Write the new state (i.e. number) to the state
  write { number: newNumber } stateRef
  -- Show the new number in the 'el' element.
  updateText (show newNumber ) el
  
-- Take a string and replace the content of an element with this string. If there is no element, do nothing.
-- Parameters:
-- str - the string representing the new content of the element
-- el - the element to be altered
updateText :: String -> Maybe Element -> Effect Unit
updateText str (Just el) = setTextContent str (toNode el)
updateText _ _ = pure unit

-- Define the 'State' that will be remembered/altered during use.
-- (We use a state of the record type, which is a little overkill here,
--  to be able to use much more complex states later.)
type State = { number :: Number }

-- A simple integer incrementer.
incrementer :: Int -> Int
incrementer number = number + 1

-- A simple integer decrementer.
decrementer :: Int -> Int
decrementer number = number - 1


main :: Effect Unit
main = do
---------------------------------------------------------------------------
-- Again, we use the same code to start:

   -- Get the body element and the (nonHTML)document
  bodyEl <- selectDivTargetFromDocument "body" =<< document =<< window
  nonHTMLdoc <- HTMLDoc.toDocument <$> (document =<< window)

---------------------------------------------------------------------------
-- *** The code for main above is the same as before. **
  -- We will use a separate, new 'main' division instead of the body as the container
  -- for our other divisions.
  mainDivElement <- createElementWithTagAndContent "div" "" nonHTMLdoc
  -- And add the created element to the body.
  maybeAppendChildNode bodyEl mainDivElement 

  -- We create a new state Reference of the type we introduced after the inputEventHandler definition.
  stateRef <- new {number: 0}

  -- First we create an element of type "button" (HTML-tag: <button>) with text "-".
  buttonElement1 <- createElementWithTagAndContent "button" "-" nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) buttonElement1

  -- Then we create an element of type "div" (HTML-tag: <div>) with the initial text: "0".
  divElement <- createElementWithTagAndContent "div" "0" nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) divElement

  -- Now we create another element of type "button" (HTML-tag: <button>), with text "+".
  buttonElement2 <- createElementWithTagAndContent "button" "+" nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) buttonElement2
  
  -- We create an eventListener for each button.
  eh1 <- eventListener (inputEventHandler decrementer (Just divElement) stateRef )
  eh2 <- eventListener (inputEventHandler incrementer (Just divElement) stateRef )

  -- And finally add those to the chosen elements.
  maybeAddEventListener eh1 (Just buttonElement1)
  maybeAddEventListener eh2 (Just buttonElement2)  
```
**Figure 1.5: Listing of code example that provides the same HTML as the first example in the Halogen Guide: two buttons for incrementing and decrementing a state counter.**

In essence, the code in Fig. 1.5 does the same as the code in Fig. 1.3. Elements are added, and an event listener is added. The only difference is that a state reference was added, which could be given to the inputEventHandler and be changed.

If you compare the result of the Fig. 1.5 code with that of the code of the first example in the Halogen Guide (https://github.com/purescript-halogen/purescript-halogen/tree/master/docs/guide), you will see that the HTML produced is exactly the same.

So, to recap, we have shown that, without using Halogen, we can provide:
* "[a] state that represents values over time", and
* "the ability to respond to DOM events (for example, when a user clicks a button)."

Which is no surprise, as Halogen itself is written in purescript.

This completes the basics for now. In the next chapter we will continue with fine-tuning with HTML element properties. [Chapter 2. HTML properties.](./Chapter2.md)

