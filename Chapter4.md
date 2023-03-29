# Purescript web programming basics tutorial: Halogen versus purescript HTML.
# Chapter 4 - Basic data storage.
In this chapter we will cover the handling of data for web programming with purescript. We introduce two new concepts: a data format and type class instances.
## Data format JSON.
To store data in files it is advisable to use a standard data format. A popular data interchange format, that has the advantage of being human-readable, is the JSON (JavaScript Object Notation, https://en.wikipedia.org/wiki/JSON) data format. One of the current standard purescript modules for this data format is argonaut (https://pursuit.purescript.org/packages/purescript-argonaut). We will use this module in our examples.

## Type class instances.
Another new concept we will be using is the type class _instance_. We refer to Chapter 6 of the Purescript by Example book for examples (https://book.purescript.org/chapter6.html). In this chapter you can read: "A type class instance contains implementations of the functions defined in a type class, specialized to a particular type." We will show application of this concept for conversions to json, and also demonstrate some examples of _Show_ type classes that are discussed in the Purescript by Example book.

## Records to data and back.
In the following we will demonstrate the conversion of a record to json, and json to string, and back. The code example below takes a record, converts it to json, converts the json to a string to be shown on a page. After that we will make a full circle by extracting the data from the string shown on the page and convert it back to a record.

**Note:** To test this code on your own machine, you will have to install the argonaut module (```spago install argonaut```).

```
module Main  where

import Prelude

import Data.Argonaut (class DecodeJson, class EncodeJson, Json, JsonDecodeError, decodeJson, encodeJson, jsonEmptyObject, parseJson, stringify, (.!=), (.:), (:=), (~>))
import Data.Either (Either(..))
import Data.Maybe (Maybe(..))
import Effect (Effect)
import Web.DOM.Document (Document, createElement)
import Web.DOM.Element (Element, toNode)
import Web.DOM.Node (appendChild, setTextContent)
import Web.DOM.ParentNode (querySelector, QuerySelector(QuerySelector))
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

-- Take a string and replace the content of an element with this string. If there is no element, do nothing.
-- Parameters:
-- str - the string representing the new content of the element
-- el - the element to be altered
updateText :: String -> Maybe Element -> Effect Unit
updateText str (Just el) = setTextContent str (toNode el)
updateText _ _ = pure unit

---------------------------------------------------------------------------
-- *** The code above is the same as before, except for some imports **
-- Next, we create a data type and define the type class instances for it for show
-- and the json encoding and decoding, and the functions to provide safe
-- default decoded data.

-- Create the data type 'Information' with a constructor and text and number content
data Information = 
  Information { text :: String 
              , number :: Int
              }

-- Create a type class instance to show the content of an Information data item.
instance showInformation :: Show Information where
  show (Information info) = "text: " <> show info.text <> ", number: " <> show info.number 

-- Create a type class instance to encode the content of an Information data item to json.
-- We use two functions from Argonaut:
-- (:=) creates a 'Tuple String Json' as key/value pair for an object.
-- (~>) extends a Json object with a 'Tuple String Json' property
instance encodeJsonInformation :: EncodeJson Information where
  encodeJson (Information info)
    -- We create a key/value pair for the json object.
    = "text" := info.text
    -- We create another key/value pair for the json object.
    ~> "number" := info.number
    -- We open the data structure with an empty json object.
    ~> jsonEmptyObject

-- Create a type class instance to decode json to an Information data item.
-- We use one new function from Argonaut:
-- (.:) which is an alias for getField.
instance decodeJsonInformation :: DecodeJson Information where
  decodeJson json = do
    -- We extract the json object.
    obj <- decodeJson json
    -- We get the field with name "text"
    text <- obj .: "text"
    -- We get the field with name "number"
    number <- obj .: "number"
    -- And return an Information record with the extracted data.
    pure $ Information { text, number }

-- Extract json information from a string.
-- We use the parseJson function from the Argonaut module of the type:
-- parseJson :: String -> Either JsonDecodeError Json
-- and bind the result to decodeJson, which will give a
-- result of type 'Either JsonDecodeError Information
tryParseJson ∷ DecodeJson Information ⇒ String → Either JsonDecodeError Information
tryParseJson stringIn = do 
  out <- decodeJson =<< parseJson stringIn
  pure out

-- Give the Information data from a decoded Json object with a possible error.
-- Give default Information data in case of a decoding error.
defaultInfoFromJson :: Either JsonDecodeError Information -> Information
defaultInfoFromJson (Left _) = Information { text: "default", number: 0 }
defaultInfoFromJson (Right json) = json

-- Create some input as information to work with
inputInformation ∷ Information
inputInformation = Information { text: "text", number: 0 }


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
  -- for our other divisions, and add three divisions to it.
  mainDivElement <- createElementWithTagAndContent "div" "" nonHTMLdoc
  -- And add the created element to the body.
  maybeAppendChildNode bodyEl mainDivElement 

  -- First we create an element of type "div" to show the input data.
  dataInputElement <- createElementWithTagAndContent "div" "input" nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) dataInputElement

  -- Then we create an element of type "div" for the JSON converted to a string.
  jsonStringElement <- createElementWithTagAndContent "div" "json as a string" nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) jsonStringElement

  -- And finally we create another element of type "div" to show the data after the full cycle of conversions.
  dataOutputElement <- createElementWithTagAndContent "div" "full circle result" nonHTMLdoc
  -- And add the created element to the new 'main' division.
  maybeAppendChildNode (Just mainDivElement) dataOutputElement

  -- Now we will demonstrate the Json conversions.
  -- First we will show the data in our record in the first element ('input').
  updateText (show inputInformation) (Just dataInputElement)

  -- Now, we will encode the Information to json.
  let encodedInformation = encodeJson inputInformation
  -- Because we cannot show json directly, we will turn it into a string using the 'stringify'
  -- function from the Argonaut module.
  let informationAsString = stringify encodedInformation
  -- And show it in the second division.
  updateText (informationAsString) (Just jsonStringElement)

  -- Now use the string extract the information (This may give a JsonDecodeError.)
  -- tryParseJson ∷ DecodeJson Information ⇒ String → Either JsonDecodeError Information
  let newTryJsonObject = tryParseJson informationAsString

  -- We will catch the JsonDecodeError and give a default Information object on an error, or
  -- the actual Information data object if there is no error.
  let newSafeExtractedInformation = defaultInfoFromJson newTryJsonObject

  -- To make full circle we will now show the record extracted from the json object.
  updateText (show newSafeExtractedInformation) (Just dataOutputElement)

  -- We return unit to close the 'do' block.
  pure unit
```
**Figure 4.1: Example code to show Json encoding an decoding.**

If you try the code example from Fig. 4.1 in https://try.purescript.org, you will see that in the json string that is shown in the second (middle) division, the order of items is reversed. This is due to the right associative definition of the (~>) combinator. This is also why the 'encodeJsonInformation' instance definition ends with the 'jsonEmptyObject'. Just read the addition of elements in reverse. It is of no consequence, because in records items are referred to by name, not order.

If you replace the informationAsString parameter with a non-Json object string, like "", in the line (line 168):
```
  let newTryJsonObject = tryParseJson informationAsString
```
For example:
```
  let newTryJsonObject = tryParseJson ""
```
You will see that the data in the output division are those of the 'default' information record.

You now have a recipe to encode records to json and decode json. The json format is very handy for storing data, because it is a standard, but also because the data remain readable for humans.

<H2> <span style= "color:green">Theory: storing data in HTML page </span></H2>

<span style= "color:green">
The following part is a theoretical experiment and is not necessary for building web pages. You can skip it without loosing the benefits of this tutorial, but it might give you ideas for your own work. I you want, you can skip to the next part.
</span>

## HTML or State monad

In the last example of Chapter 1 we introduced a global mutable state using the Effect.Ref module (https://pursuit.purescript.org/packages/purescript-refs/6.0.0/docs/Effect.Ref#v:new). We used this state to store data. This monad was necessary to model a mutable state in pure code. However, the systems we build using purescript and HTML are not pure.

### HTML + purescript and purity
Purescript uses the Effect monad to handle native side-effects. And, for example, DOM manipulation is handled in this effect monad as we have seen in all our previous examples. This suggests that it might be possible to use the HTML page as storage. However, if we would want to store data for our code on a HTML page, it would be distracting or confusing for users to see this stored data. So it would be preferable to write the information out of view.

Actually, the above example in Fig. 4.1 contains all the items to prove this. The only thing we have to change is to hide the second element, the 'jsonStringElement'. This element contains json data converted to a string. In fact, this conversion to a string is a method to store or transmit data

Hiding an element in HTML is a matter of setting the display style to 'none'. And the display style is an attribute. So, instead of setting the element style to 'red' as we did in Part 2: Attributes:
```
_ <- DOMElement.setAttribute "style" "color:red" createdElement
```
We set the style to "display:none". 

You can try this in the above code in Fig. 4.1 (in https://try.purescript.org, or your own system) by adding the import of the Web.DOM.Element module:
```
import Web.DOM.Element as DOMElement
```
and adding the following code after the creation and appending of the 'jsonStringElement':
```
_ <- DOMElement.setAttribute "style" "display:none" jsonStringElement
```
In our example we now have:
- the creation of a record, created with:  ```inputInformation = Information { text: "text", number: 0 }```
- conversion of the record to json with: ```let encodedInformation = encodeJson inputInformation ```
- conversion of the json data to a string, which is a data storage format: ```let informationAsString = stringify encodedInformation```
- (hidden) storage of the data on the page with: ```updateText (informationAsString) (Just jsonStringElement) ```

All we need is a method to recover the data (in String format) from the storage element. 

There are libraries that provide easy access to these dom-data. However, our aim is to provide only examples that may be tried in Try Purescript (https://try.purescript.org), and these libraries (Nonbili and Specular) are not part of the standard repository of Try Purescript. You are invited to complete this theoretical experiment on your own machine, but our example stops here.

This ends the theoretical part of this chapter, but the next chapter continues with a very practical subject: canvas elements and images. [Chapter 5. Canvas and images.](./Chapter5.md)