# Purescript web programming basics tutorial: Halogen versus purescript HTML. 
# Chapter 2 - HTML properties.
 This is the second chapter of a tutorial that meant as an aid for people who wish to develop web HTML using purescript. In Chapter 1 we showed how to build the essential, basic parts of an HTML page using basic purescript HTML, without Halogen. We gave all the tools to recreate the first example in the Halogen Guide (https://github.com/purescript-halogen/purescript-halogen/tree/master/docs/guide). Now, we will continue to provide alternatives to the next part discussed in the Halogen Guide: How to set properties of HTML elements.

## HTML element properties
For your convenience, we copied the relevant code from the Halogen Guide here:
```
import Halogen.HTML as HH
import Halogen.HTML.Properties as HP

html =
  HH.div
    [ HP.id "root" ]
    [ HH.input
        [ HP.placeholder "Name" ]
    , HH.button
        [ HP.classes [ HH.ClassName "btn-primary" ]
        , HP.type_ HP.ButtonSubmit
        ]
        [ HH.text "Submit" ]
    ]

```
**Figure 2.1 Code copied from the Halogen Guide example on properties**

Unfortunately, the Halogen Guide does not provide a complete piece of code to try in https://try.purescript.org. So, there is no opportunity to easily compare. In our opinion this is a disadvantage of Halogen: you will always need at least a template to test software.
For our purescript HTML code we will provide complete, testable code. So you can see and try for yourself.

We can see that the following properties are set:
- id
- placeholder - for an input element
- classname

Note that the placeholder is added only within a separate tag (HH.input) and that the classname is also added only after adding the tag HP.classes. We point this out, because in our code example you will see that this is just a manner for hiding the type(s) (conversions) needed.
For our example we will base our code on the code from Fig. 1.2 in Chapter 1 to create a basic html element on a page. So the code up to the line is the same, except for some imports.
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
import Web.DOM.Element as DOMElement

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
  createdElement <- createElementWithTagAndContent "input" "new div" nonHTMLdoc
  -- And append it to the body.
  maybeAppendChildNode bodyEl createdElement

  -------------------------------------------------------------------------------
  -- The above code is the same as in Fig. 2 of Chapter 1. Except for the element tag
  -- of the created element in the call of the createElementWithTagAndContent function.
  -- We need to use the 'input' tag to see the placeholder information.
  
  -- To add attributes there are often several ways. We will show two ways using
  -- the Web.DOM.Element module imported as DOMElement. The first is by just
  -- setting an attribute using the attribute name. The second is by using a 
  -- dedicated function.
  -- First we will us the attribute name:
  -- Set id:
  _ <- DOMElement.setAttribute "id" "root" createdElement
  -- Add a placeholder:
  _ <- DOMElement.setAttribute "placeholder" "Name" createdElement
  -- Add a classname:
  _ <- DOMElement.setAttribute "classname" "btn-primary" createdElement

  -- You may do the same for id and classname as follows:
  _ <- DOMElement.setId "root2" createdElement
  _ <- DOMElement.setClassName "btn-primary2" createdElement
  -- But setPlaceHolder is not part of the Web.DOM.Element module.

  -- The last statement in a 'do' block must be an expression. We just return 'unit' for now.
  pure unit

```
**Figure 2.2 Code for setting HTML attributes with purescript, without Halogen.**

You may try the above code in https://try.purescript.org, but you will only see the effect of the placeholder. If you are familiar with HTML coding in your browser, you can inspect the element and see that its attributes are as expected. Because we changed the id and classname twice in the example, the browser will only show the last versions ("root2" and "btn-primary2").

As you can see, adding HTML attributes is pretty straightforward, if you use the purescript Web.Dom.Element purescript module. You could, for example, try:
```
_ <- DOMElement.setAttribute "style" "color:red" createdElement
```

We leave it up to you to compare the Halogen code and our way. Both have their advantages. Our way may be easier to adapt to for those used to programming javascript.

**Note:** The Halogen Guide mentions that they use className as a newtype. In Chapter 14 of the purescript book (https://book.purescript.org/chapter14.html) HTML data types are discussed, and an example is given on how to handle attributes:
```
newtype Element = Element
  { name         :: String
  , attribs      :: Array Attribute
  , content      :: Maybe (Array Content)
  }

  ....

newtype Attribute = Attribute
  { key          :: String
  , value        :: String
  }
```
**Figure 2.3 Partial code for HTML data type from Chapter 14 of the purescript book.**

 So, please consider using newtypes when you apply our setAttribute method in Fig. 2.2.

 In the example in Fig. 2.2 we showed how to create a placeholder to aid the user. We expect the user to input some information, guided by this placeholder. The input by the user needs to be handled. This is where event handling comes into play. A basic form of event handling was already discussed in Chapter 1, but in the next chapter we will delve deeper into the handling of events with purescript. [Chapter 3. Events.](./Chapter3.md)




