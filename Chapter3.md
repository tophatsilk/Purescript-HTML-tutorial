[Introduction](./Introduction.md) | [Chapter 1. The basics.](./Chapter1.md) | [Chapter 2. HTML properties.](./Chapter2.md) | [Chapter 3. Events.](./Chapter3.md) | [Chapter 4. Basic data storage.](./Chapter4.md) | [Chapter 5. Canvas and images.](./Chapter5.md)
# Purescript web programming basics tutorial: Halogen versus purescript HTML.
# Chapter 3 - Events.
In Chapter 1 we ended with a clickable button with eventType "click":
```
inputEvent :: EventType
inputEvent = EventType ("click")
```
And we already hinted to other types. In Chapter 2 we showed the HTML type 'input', but it was just a passive input box. So let's make an input box that shows interaction. 

## The input event type
We will make an input box that triggers an event upon input, and use that event to alter the HTML page, using information from that event.

This chapter is based on the excellent tutorial of Kevin B. Greene, "Functional Programming for the Web: PureScript Foreign Import and DOM Events" (https://medium.com/@KevinBGreene/functional-programming-for-the-web-purescript-foreign-import-and-dom-events-8c76f6f5a16e). 
We adapted the code to be consistent with our earlier code.

```
module Main where

import Prelude

import Data.Maybe (Maybe(..))
import Effect (Effect)
import Web.DOM.Document (Document, createElement)
import Web.DOM.Element (Element, setAttribute, toEventTarget, toNode)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
import Web.Event.Event (Event, EventType(EventType), target)
import Web.Event.EventTarget (EventListener, addEventListener, eventListener)
import Web.Event.Internal.Types (EventTarget)
import Web.HTML (window)
import Web.HTML.HTMLDocument as HTMLDoc
import Web.HTML.HTMLInputElement (HTMLInputElement, fromEventTarget, value)
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
createElementWithTagAndContent ∷ String → String → Document -> Effect Element
createElementWithTagAndContent tag contents nonHTMLdoc = do
  -- Create the new element
  createdElement <- createElement tag nonHTMLdoc
  -- Add its contents (no return needed, hence the anonymous function '_'.)
  _ <- setTextContent contents (toNode createdElement)
  pure createdElement

-- Append a child if a parent element is given.
-- The function appendChild from Web.DOM.Node takes nodes as arguments 
-- (appendChild :: Node → Node → Effect Unit), so the elements need to be
-- converted to nodes with toNode from Web.DOM.Element.
maybeAppendChildNode ∷ Maybe Element → Element → Effect Unit
maybeAppendChildNode (Just parent1) child1 = appendChild (toNode child1) (toNode parent1)
maybeAppendChildNode _ _ = pure unit

-- Add an eventListener, if an element is provided.
-- Parameters:
-- eh - the EventListener to be used.
-- el - the element to be equipped with the listener.
maybeAddEventListener ∷ EventListener → Maybe Element → Effect Unit
maybeAddEventListener eh (Just el) = addEventListener inputEvent eh true (toEventTarget el)
maybeAddEventListener _ _ = pure unit

-- Take a string and replace the content of an element with this string. If there is no element, do nothing.
-- Parameters:
-- str - the string representing the new content of the element
-- el - the element to be altered
updateText :: String -> Maybe Element -> Effect Unit
updateText str (Just el) = setTextContent str (toNode el)
updateText _ _ = pure unit

---------------------------------------------------------------------------
-- ** The code above is the same as before, except for some imports **
-- Next, we introduce some new functions to handle the input event and its imput.

--Define the EventType. In this case: "input". The type is the only thing we change.
inputEvent :: EventType
inputEvent = EventType ("input")

-- Get the content of an HTMLInputElement as a string. (We use 'HTMLInputElement' and 'value'
-- from the Web.HTML.HTMLInputElement module.)
inputValue ∷ Maybe HTMLInputElement → Effect String
inputValue (Just el) = value el
inputValue _         = pure ""

-- Get the content of an EventTarget as an Effect String
targetValue ∷ Maybe EventTarget → Effect String
targetValue (Just et) = inputValue (fromEventTarget et)
targetValue _         = pure ""

-- Use the above targetValue function to obtain the content of the target element using the event.
-- The 'target' function from 'Web.Event.Event' is of type: Event -> Maybe EventTarget)
eventValue ∷ Event → Effect String
eventValue evt = targetValue (target evt)

-- Perform this action when the event is triggered:
-- Parameters:
-- el  - the element to which the new text will be sent.
-- evt - the event
inputEventHandler ∷ Maybe Element -> Event → Effect Unit
inputEventHandler el evt = do
  -- Get the value of the input.
  str <- eventValue evt
  -- Send it to the output element.
  updateText str el


main :: Effect Unit
main = do
---------------------------------------------------------------------------
-- Again, we use the same code to start:

   -- Get the body element and the (nonHTML)document
  bodyEl <- selectDivTargetFromDocument "body" =<< document =<< window
  nonHTMLdoc <- HTMLDoc.toDocument <$> (document =<< window)

---------------------------------------------------------------------------
-- *** The code for main above is the same as before. **
  -- As before, we will use a separate, new 'main' division instead of the body 
  -- as the container for our other divisions.
  mainDivElement <- createElementWithTagAndContent "div" "" nonHTMLdoc
  -- And add the created element to the body.
  maybeAppendChildNode bodyEl mainDivElement 

  -- First we create an element of type "div" with text.
  outputElement <- createElementWithTagAndContent "div" "This text will change, when you type below." nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) outputElement

  -- Then we create an element of type "input".
  inputElement <- createElementWithTagAndContent "input" "Type here" nonHTMLdoc
  -- Remember the 'placeholder' attribute from Chapter 2:
  _ <- setAttribute "placeholder" "You can type here." inputElement
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) inputElement
  
  -- We create an eventListener for the input, and refer to the output element
  -- so the inputEventHandler knows where to put the output.
  eh <- eventListener (inputEventHandler (Just outputElement))  

  -- And finally add it to the input element.
  maybeAddEventListener eh (Just inputElement)
```
**Figure 3.1 Code example that writes the input from an input element to another element.**

If you try the above code of Fig. 3 in https://try.purescript.org. You will see a division with text and, below it, an input element. When you type in the input element, the typed text will appear in the upper text box.

To do this, we changed the event type we saw earlier in Chapter 1, from "click" to "input". To use the input we had to capture the input from the event (with the eventValue function), finding the event target and its value. We need an HTMLInputElement to extract the string, so we use fromEventTarget (EventTarget -> Maybe HTMLInputElement) to convert it in the targetValue function.

There is an important difference between this example and the button example in Fig 1.5 in Chapter 1. We did not simply reacted to an event, but also used the information from the event target.

Event handling plays a key role in web page programming.

## HTML events
HTML programming with javascript relies heavily on events. (See, for example this tutorial: https://www.w3schools.com/js/js_events.asp.)

For example, if we look back to the first Halogen example in the Halogen Guide (https://github.com/purescript-halogen/purescript-halogen/tree/master/docs/guide), we see that the first step of the main function calls a function called HA.awaitBody:
```
main :: Effect Unit
main = HA.runHalogenAff do
  body <- HA.awaitBody
  ...
```
The description in pursuit (https://pursuit.purescript.org/packages/purescript-halogen/7.0.0/docs/Halogen.Aff.Util#v:awaitBody) simply states: "Waits for the document to load and then finds the body element." If we dig a little deeper, we see in the source code a call to the awaitLoad function. This function adds an eventListener for the eventType 'domcontentloaded' from the Web.HTML.Event.EventTypes module.

Waiting for the event that is triggered when the document is completely loaded, is standard practice in javascript coding (See, for example, the jQuery ready method: https://www.w3schools.com/jquery/event_ready.asp). It is, after all, a logical point to start reacting to other events on the page.

In our example in Fig 3.1 above we had the user input information on the page. We used this information immediately to alter the web page itself. But often information gathered or created by a web page will need to be stored or transferred to another location or page. This is where the json data format we will discuss in the next chapter comes in handy. [Chapter 4. Basic data storage.](./Chapter4.md)