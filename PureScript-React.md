# How to write PureScript React to replace TypeScript React in 2021

I refactored several thousand lines of TypeScript into PureScript, and here are my notes
about the tricky bits. I say “refactor” instead of “rewrite,” because
the word “rewrite” sometimes causes too much excitement.
Anyway, “refactor” is accurate. A TypeScript program contains
much more than TypeScript, it also consists of a whole build system
with bundlers, a deployment system, a backend, assets, et cetera. We get to keep
all of that other stuff. We’re just going to “refactor” one of the passes
of the JavaScript transpiler process, and I recommend that’s how you should
frame it when you’re describing this process to the impatient.

### On TypeScript

TypeScript is an example of family of languages which John Backus characterized as
[“fat and weak”](https://dl.acm.org/doi/pdf/10.1145/359576.359579). He included
his own invention, FORTRAN, in this family.

The fatness of TypeScript is apparent in its complicated specification and
noisy syntax. The weakness is
apparent when we try to write useful programs with TypeScript, and discover that it’s
basically impossible, and then we resort to
[external domain-specific languages](https://en.wikipedia.org/wiki/Domain-specific_language#External_and_Embedded_Domain_Specific_Languages)
because we can’t express what we want to express in TypeScript.
That's why a typical TypeScript React program will usually contain the following
external domain-specific languages:

* *TSX*
* *regular expressions*
* *styled-components* or *emotion* templates

Also, the TypeScript type system is underpowered and has too many escape 
hatches. It’s been my
observation that when TypeScript programmers are passing a string to a function
they will dutifully annotate it as type `:string`, but when the types get
difficult they give up and pass `:any`. It’s in exactly those difficult
situations that we need the compiler’s help checking the type.

### On PureScript

What is [PureScript](https://www.purescript.org/)?
It is a dialect of [Haskell](https://www.haskell.org/).

There are some slight differences.
A lot of people, myself included, think that PureScript records are
an improvement over Haskell records. But they are the same language.
`purs` is now the second-most-popular Haskell compiler implementation,
after `ghc`.

PureScript, being relatively lean and powerful, won’t need any
external domain-specific languages. We’ll just express our program in
plain PureScript.

### On PureScript React Basic Hooks

If you want to write web applications in 2021, then
PureScript React Basic Hooks is a very good framework choice.

There are many other PureScript immediate-mode GUI libraries. Most of them
were designed by Phil Freeman. React Basic Hooks is the best one.

The most noticeable thing about __purescript-react-basic-hooks__ is the
[`Hook`](https://pursuit.purescript.org/packages/purescript-react-basic-hooks/docs/React.Basic.Hooks#t:Hook) indexed monad.

To understand why `Hook` is an
[indexed monad](https://qiita.com/kimagure/items/a0ee7313e8c7690bf3f5),
please read this short dramatization
of a conversation which I had at Shake Shack at the International Forum in Tokyo.

__Me:__
I want to loop over some `Hook`s, but
I can’t find [`traverse`](https://pursuit.purescript.org/packages/purescript-foldable-traversable/docs/Data.Traversable#v:traverse) for the `Hook` monad and the compiler won’t let me write it.
What gives?

__[Robert Porter](https://github.com/robertdp):__
Calling a `Hook` in a loop is forbidden by the
[*Rules of Hooks*](https://reactjs.org/docs/hooks-rules.html#only-call-hooks-at-the-top-level).
That is why `Hook` is an indexed monad; because 
Madeline Trotter and Phil
Freeman noticed that
the algebraic structure of the indexed monad matches neatly with
the *Rules of Hooks*, so they created the `Hook` indexed monad.
In TypeScript the only way to stop users from violating
the *Rules of Hooks* is by linting or scolding. In PureScript React Basic
Hooks, a *Rules of Hooks* violation is a compile-time type error.

__Me:__ *jaw slackens on my Shack Stack* That makes so much sense.

### Our strategy for re-writing the whole program

When rewriting a computer program in another language, one often must
simply rewrite the whole program and it won't be done until it’s done.

In some lucky cases, there exists good FFI bindings between the source
and target languages. Then it’s possible to swap out parts of the old
program and replace them with parts written in the new language. The key is
to find good “parts,” to find boundaries out of which a section of the program
can be cleanly pried and replaced, [ship-of-Theseus](https://en.wikipedia.org/wiki/Ship_of_Theseus)-style.

For re-writing from TypeScript React to PureScript React Basic Hooks, the
situation is very lucky, because we have very natural clean boundaries for
replacement: the React components.

We will refactor the TypeScript into PureScript one React component
at a time, and our ship will remain seaworthy at every step.

The most mentally taxing programming will be at the interoperation boundary
between PureScript and TypeScript. There’s a lot of boilerplate code at
that boundary, and since we're jumping between type systems, the compiler
can’t help us with typechecking. For that reason, we’ll want to pry
out sections larger than a single React component as we gain momentum on
our refactor. Pick a top-level React component to replace
and then recursively replace of all of the components it depends on, in
a single step. We can pretty much go file-by-file, and line-by-line.

If we have file named `TopPage.tsx`, then we can create a `TopPage_.purs`
right next to it, and start writing. We would be able to create `TopPage.purs`
*except* that we might want some
[foreign imports, in which case we would need `TopPage.js`](https://github.com/purescript/documentation/blob/master/language/FFI.md),
which would conflict with `TopPage.tsx` while bundling.
`TopPage_.js` will not conflict.


## PureScript-TypeScript interop

Notes on how to write the interface between PureScript and TypeScript.

Our point of interface will almost always be the
[`ReactComponent`](https://pursuit.purescript.org/packages/purescript-react-basic-hooks/docs/React.Basic.Hooks#t:ReactComponent). We want to call PureScript React components from
TypeScript, and vice versa.

### PureScript FFI

We also must understand how FFI works in PureScript.

[github.com/purescript/documentation/blob/master/language/FFI.md](https://github.com/purescript/documentation/blob/master/language/FFI.md)

[github.com/purescript/documentation/blob/master/guides/FFI.md](https://github.com/purescript/documentation/blob/master/guides/FFI.md)

### Thomas Honeyman’s article

[How to Write PureScript React Components to Replace JavaScript](https://thomashoneyman.com/articles/replace-react-components-with-purescript/)
and discussion:
[discourse.purescript.org/t/updated-how-to-replace-react-components-using-purescripts-react-libraries/](https://discourse.purescript.org/t/updated-how-to-replace-react-components-using-purescripts-react-libraries/)

### TypeScript [Union Types](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#union-types)

There is no equivalent PureScript built-in feature which compiles to the same runtime representation.
But these libraries can help.

[github.com/natefaubion/purescript-variant](https://github.com/natefaubion/purescript-variant)

[github.com/jvliwanag/purescript-untagged-union](https://github.com/jvliwanag/purescript-untagged-union)

[github.com/paluh/purescript-undefined-is-not-a-problem](https://github.com/paluh/purescript-undefined-is-not-a-problem)

[github.com/doolse/purescript-tscompat](https://github.com/doolse/purescript-tscompat)

OhYes

[github.com/justinwoo/purescript-ohyes](https://github.com/justinwoo/purescript-ohyes)

[OhYes, you can interop with TypeScript using PureScript](https://qiita.com/kimagure/items/4847685d02d4b15a556c)

### TypeScript [String Literal](https://www.typescriptlang.org/docs/handbook/literal-types.html#string-literal-types) Union Types

```typescript
type Alignment = "left" | "right" | "center"

```

This is common idiom in TypeScript. The runtime representation of these things
is just a string. How do we make
an equivalent thing in PureScript with the same runtime representation? It’s
a puzzle.

Maybe with [`Symbol`](https://pursuit.purescript.org/packages/purescript-prelude/4.1.1/docs/Data.Symbol#t:SProxy), the PureScript type-level string.

Or [github.com/natefaubion/purescript-variant](https://github.com/natefaubion/purescript-variant).


### TypeScript [Intersection Types](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#intersection-types)

```typescript
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

type ArtworksResponse = ArtworksData & ErrorHandling;
```

We can use PureScript [Row Polymorphism](https://github.com/purescript/documentation/blob/master/language/Types.md#row-polymorphism) to create equivalent PureScript types
with the same runtime representation.

```purescript
type ErrorHandlingRow r =
  ( success: Boolean
  , error: Nullable { message :: String }
  | r
)

type ArtworksDataRow r =
  ( artworks :: Array { title :: String }
  | r
  )

type ArtworksResponse = Record (ArtworksDataRow (ErrorHandlingRow ()))
```

## Calling TypeScript React components from PureScript

Suppose we have this TypeScript React component, and we want to wrap it
so that we can call it from PureScript.

__src/Tags.tsx__
```typescript
export interface Props_tags {
  tags: [string]
}
export default (props:Props_tags) => {
...
}
```

To wrap the foreign `Tags` component in PureScript,
create files `Tags_.purs` and `Tags_.js`.

__src/Tags_.purs__
```purescript
module Tags (tsxTags) where

import React.Basic (ReactComponent)

-- | Must agree with the TypeScript `interface Props_tags`.
type Props_tags =
  { tags :: Array String
  }

foreign import tsxTags :: ReactComponent Props_tags
```

__src/Tags_.js__
```javascript
"use strict";

exports.tsxTags = require('src/Tags').default;
```

(The `tsx` prefix convention makes it easy to see which components in `.purs`
files are foreign.)

Then we can use the foreign component:

```purescript
import React.Basic.DOM (div_)
import React.Basic.Hooks (element)
import Tags (tsxTags)
...
div_ [ element tsxTags {tags:["one"]} ]
```

### React diffing algorithm and `foreign import`

More helpful advice from [Robert Porter](https://github.com/robertdp):

The [React diffing algorithm uses referential equality for the props](https://dev.to/tylerthehaas/referential-equality-in-react-127h).

If a PureScript component has class constraints, then
the constraint dictionary will get passed on every render, and React will consider the constraint dictionary to
be part of the props. A new constraint dictionary object will be created on each call, so the constraint dictionary will
not compare equal to the contraint dictionary from the last call, which will cause
the React diffing algorithm to re-render the component on every render.

If the component reference it returns is the same every time, then the problem won’t occur.
You are more likely to run into this problem with a component written in PS that uses constraints, because the compiled JS will return a new component reference every time it runs.

The way around this is by making a local wrapper alias that fixes all the constraints to known types, thus avoiding the problem of re-evaluation.

From React’s point of view, constrained components in the JSX are acting kind of like `Effect`. So it can return a “new” component every time. So either satisfy the constraints before using it in the JSX, or make sure that it’s a “pure” effect that returns the same (referentially equal) value every time.

A `foreign import` `ReactComponent` will return the same (referentially equal) value every time.

## Calling PureScript React components from TypeScript

I like to use the top-level `unsafePerformEffect` technique for creating exportable PureScript
React components, even though Madeline [“wouldn't say it’s the right thing to do.”](https://github.com/spicydonuts/purescript-react-basic-hooks/issues/41)

### Ambient Definition file

We have a `types/purs` directory for TypeScript definition files.

For every `MyModule.purs` which exports a `ReactComponent` named `psxThing` in a `MyModule` module, we'll need to create
a `types/purs/MyModule.d.ts` which declares a TypeScript module.

There are [two types of React components](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/bab0a49d79fb3cd850db3174d0ed91a85be7f433/types/react/index.d.ts#L82): `ComponentClass`, which is the “traditional” “classic” class-based
component, and `FunctionComponent`, which is the Hooks-based component type.


We’ll use the type [`FunctionComponent`](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/a17292911589e315b72ca8034cb9c8ac5eff4030/types/react/v16/index.d.ts#L546), because that corresponds to [the
`ReactComponent` type returned](https://github.com/spicydonuts/purescript-react-basic-hooks/blob/5e43ca349a960c238e88219d0121603ae0c9889b/src/React/Basic/Hooks.purs#L136) by
[`React.Basic.Hooks.reactComponent`](https://pursuit.purescript.org/packages/purescript-react-basic-hooks/6.2.0/docs/React.Basic.Hooks#v:reactComponent).


https://www.typescriptlang.org/docs/handbook/modules.html#ambient-modules

__`types/purs/MyModule.d.ts`__

```typescript
declare module 'purs/MyModule' {
  import { I_Props } from "propsinterface"; // Do any TypeScript imports here
  const psxThing : React.FunctionComponent<I_Props>;
}
```

### TSX

TSX components names must have an uppercase first letter. PureScript
component names must have a lowercase first letter.
Our convention is that in a `.tsx` file, we import PureScript components
like `psxThing` and then alias them.

```typescript
import {psxThing as PSXThing} from 'purs/MyModule';

<PSXThing/>
```

Then we can look at `.tsx` files and see how many of the components are `PSX`.

And when we're finished, then we can change all of the `psx* :: ReactComponent`
FFI components into native React Basic Hooks `psx* :: Component`.

### react-router and `history.push()`

We published
[__purescript-react-basic-router__](https://github.com/xc-jp/purescript-react-basic-router)
so that we can `push` to a __react-router-dom__ `History` object.

### How to `getElementById`

A bit tricky, so here is [the trick](https://lobste.rs/s/wa99yt/coming_purescript_from_haskell_reflex#c_faof1j):

```purescript
import Web.DOM.Document (toNonElementParentNode)
import Web.DOM.NonElementParentNode (getElementById)
import Web.HTML (window)
import Web.HTML.HTMLDocument (toDocument)
import Web.HTML.Window (document)

do
  rootElMaybe <- getElementById "root" =<< toNonElementParentNode <$> toDocument <$> (document =<< window)
```

### React Transition Group

The whole point of
[React Transition Group](https://reactcommunity.org/react-transition-group/)
is that when we want to trigger a CSS animation right before a component gets
unmounted, then we want to wait for the CSS animation to finish before we
unmount the component. That's the essential feature.

I recommend using
[`Effect.Aff.delay`](https://pursuit.purescript.org/packages/purescript-aff/6.0.0/docs/Effect.Aff#v:delay)
to get the same feature.

Suppose we want to remove an icon after a 20-second vanishing animation.

```purescript
React.do

  icon /\ setIcon <- useState true
  iconOpacity /\ setIconOpacity <- useState "1"

  let iconVanish :: Effect Unit
      iconVanish = do
        setIconOpacity $ const "0"
        launchAff_ do
          delay $ Milliseconds 20000.0
          liftEffect $ setIcon $ const false

  pure $ if icon
    then R.img
      { style: R.css
        { opacity: iconOpacity
        , transition: "opacity 20s ease-in"
        }
      }
    else empty
```

I've been advised by my colleague that this is not a good technique, because
the whole component might get unmounted while the `delay` is waiting,
and then `setIcon`  will be called on the unmounted component. So maybe
`useAff` instead of `launchAff_` would be better here.

### Foreign

Question: How do we read properties of a plain JSON object which has been passed to us
when we are uncertain of the structure of the JSON object?

Answer: The [`F` monad](https://pursuit.purescript.org/packages/purescript-foreign/6.0.0/docs/Foreign#t:F). [Parse, don’t validate.](http://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)

If we have a PureScript data type which we want to translate the JSON object into, then
we can use
[`Simple.JSON.read'`](https://pursuit.purescript.org/packages/purescript-simple-json/8.0.0/docs/Simple.JSON#v:read').
to automatically parse the JSON object into our PureScript type.

## vscode

I recommend these extensions:

* PureScript IDE
* PureScript Language Support
* Vim
* Remote - SSH
* GitLens
* Dhall Language Support
* Nix Environment Selector

## Image inlining

In most bundlers, there is a technique by which one can `import` an image as a `string`,
so that it gets compiled into inline JavaScript which looks like this:

```javascript
var spinner = "data:image/png;base64,iVBORw0KGgoAAA.....";
```

This works, for example, with [rollup.js](http://rollupjs.org) and an appropriately configured `@rollup/plugin-url`.

For this TypeScript:

```typescript
import spinner from 'assets/spinner.png';
```

We can accomplish the same thing in PureScript like this:

__Assets.purs__

```haskell
module Assets where
foreign import spinner :: String
```

__Assets.js__

```javascript
"use strict";
exports.spinner = require('assets/spinner.png').default;
```

## CSS

When we want to write inline CSS instead of a stylesheet but we also want to use
[CSS selector combinators](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors#combinators),
then we will want to use the older
[__styled-components__](https://styled-components.com/)
or the newer, better
[__emotion__](https://emotion.sh/).

__styled-components__ is not available in PureScript React Basic, but
[__purescript-react-basic-emotion__](https://pursuit.purescript.org/packages/purescript-react-basic-emotion) is available, and very good. The syntax
and behavior of __styled-components__ and __emotion__ is almost exactly the
same, so it’s easy to
refactor TypeScript with __styled-components__ into PureScript with
__emotion__.

## Library substitutions

| | TypeScript | PureScript |
|-|------------|------------|
| `XMLHTTPRequest` | [__axios__](https://www.npmjs.com/package/axios) | [__affjax__](https://pursuit.purescript.org/packages/purescript-affjax) |
| CSS class generation | [__styled-components__](https://styled-components.com/) | [__react-basic-emotion__](https://pursuit.purescript.org/packages/purescript-react-basic-emotion) |
| React router | | [__react-basic-router__](https://github.com/xc-jp/purescript-react-basic-router) |
| String interpolation | | [__interpolate__](https://pursuit.purescript.org/packages/purescript-interpolate/) |
| Loader for WebPack | | [__purs-loader__](https://github.com/ethul/purs-loader) [__craco-purscript-loader__](https://github.com/andys8/craco-purescript-loader) |
| Rollup Plugin | | [__rollup-plugin-purs__](https://github.com/Pauan/rollup-plugin-purs) |
