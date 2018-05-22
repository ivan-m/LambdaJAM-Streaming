---
title: I Streamed a Stream
author: Ivan Lazar Miljenovic
date: 23 May, 2018
...

Why do this Code-Jam?
=====================

What's wrong with the status-quo?
---------------------------------

> * Monads are great for specifying I/O.
> * But default lazy I/O in Haskell is broken.
>     - (Except in certain specific cases.)
>     - (Maybe not broken, but difficult to get right.)

What's the solution?
--------------------

. . .

Stream Processing!

. . .

... no, not parallel programming ...

. . .

... and not just for Haskell ...

* * *

> [...] P J Landin's original use for streams was to model the
> histories of loop variables, but he also observed that streams could
> have been used as a model for I/O in ALGOL 60.
>
> -- _A Survey Of Stream Processing, R. Stephens, 1995_

* * *

> _Stream processing_ defines a pipeline of operators that transform,
> combine, or reduce (even to a single scalar) large amounts of
> data. Characteristically, data is accessed strictly linearly rather
> than randomly and repeatedly -- and processed uniformly. The upside
> of the limited expressiveness is the opportunity to process large
> amount of data efficiently, in constant and small space.
>
> -- _Oleg Kiselyov_, <http://okmij.org/ftp/Streams.html>

Possible Implementations
------------------------

> * Iteratees (Oleg Kiselyov, 2008)
> * `iteratee` (John Lato, 2009 -- 2014)
> * `enumerator` (John Millikin, 2010 -- 2011)
> * `conduit` (Michael Snoyman, 2011 --)
> * `pipes` (Gabriel Gonzalez, 2012 --)

* * *

> * `machines` (Edward Kmett, 2012 --)
> * `io-streams` (Gregory Collins, 2013 --)
> * `quiver` (Patryk Zadarnowski, 2015)
> * `streaming` (Michael Thompson, 2015 --)
> * `streamly` (Harendra Kumar, 2017 --)

Why `streaming`?
----------------

> * It's simpler!
> * `Stream (Of a) m r ≈ m ([a], r)`
> * Standard function composition! No need for new operators!
> * Gaining popularity for cases not needing extra power of pipes,
>   conduit, etc.

A nice example
--------------

```{.haskell style="font-size:70%"}
-- | Once authenticated, send the data through to the API.
sendData :: Client UploadAPI -> Options -> IO ()
sendData f opts =
  withBinaryFileContents (dataFile opts) $
    withErr
    . withClientErrors     -- Handle errors from sending the data
    . tryStreamData f opts -- Send data to API
    . withClientErrors     -- Handle errors from parsing the CSV
    . transformData        -- Convert the CSV values to what we need
    . decodeByName         -- Convert the file contents into DBData
  where
    -- | If some high-level CSV parsing exception occurs, print it.
    withErr :: ExceptT CsvParseException IO () -> IO ()
    withErr = (either (liftIO . print) return =<<) . runExceptT
```

A not-so-nice example
---------------------

```{.haskell style="font-size:70%"}
-- | Take a stream of values and convert it into a stream of streams,
--   each of which has no two values with the same result of the
--   provided function.
disjoint :: forall a b m r. (Eq b, Hashable b, Monad m)
            => (a -> b) -> Stream (Of a) m r
            -> Stream (Stream (Of a) m) m r
disjoint f = loop
  where
    -- Keep finding disjoint streams until the stream is exhausted.
    loop stream = S.effect $ do
      e <- S.next stream
      return $ case e of
                 Left r -> return r
                 Right (a, stream') -> S.wrap $
                   loop <$> (S.yield a *> nextDisjoint (f a) stream')

    -- Get the next disjoint stream; i.e. split the stream when the
    -- first duplicate value is found.
    --
    -- Provided is an initial seed for values to be compared against.
    nextDisjoint :: b -> Stream (Of a) m r
                    -> Stream (Of a) m (Stream (Of a) m r)
    nextDisjoint initB = S.breakWhen step (False, HS.singleton initB)
                                     fst id
      where
        -- breakWhen does the test /after/ the step, so we use an
        -- extra boolean to denote if it's broken.
        -- PRECONDITION: before calling set, the boolean is True
        step (_, set) a = (HS.member b set, HS.insert b set)
          where
            b = f a
```

Exercises
=========

Where to get them?
------------------

```bash
git clone https://github.com/ivan-m/LambdaJAM-Streaming-exercises.git
```

How to do them?
---------------

Look at that convenient README!

Exercise 0
==========

Build yourself a Stream!
------------------------

Time for some basic Haskell!

. . .

... does anyone want to do this?

Results
-------

> * Basic understanding of the `Stream` type
>     - Lists interspered with monads!
> * How to manually decompose it
> * Hopefully a desire not to manually decompose in practice.

Exercise 1
==========

Functor Streaming
-----------------

What's with that `f` in `Stream f m r`?

Results
-------

> * Know why `Stream` can take any functor
> * Basic understanding of how `Stream`s can contain other `Stream`s
> * How `Stream`s compare to pipes and conduits.

Exercise 2
==========

Streams of Streams
------------------

* `Stream f m (Stream f m r)`{.haskell}
    - Leftovers (ala Conduit)
* `Stream f (Stream g m) r`{.haskell}
    - A stream created as a Monadic effect
* `Stream (Stream f m) m r`{.haskell}
    - An actual stream of streams (e.g. grouping).
* `Stream (Of (Stream f m v)) m r`{.haskell}
    - Don't think this is useful
* `Stream (ByteString m) m r`{.haskell}
    - Using `streaming-bytestring`.

Results
-------

> * Know why we might want to compose `Stream`s in different ways.

Exercise 3
==========

Streams are powerful lists
--------------------------

. . .

Including the infamous infinite Fibonacci definition!

Results
-------

> * I can stop telling you how `Stream`s are list-like.

Exercise 4
==========

How to do I/O
-------------

The whole reason we started this!

Results
-------

> * How to do I/O with `Stream`s.
> * Resource management best practices.

Exercise 5
==========

Tying it all together!
----------------------

Try to actually do something with what you've (hopefully) learnt
today!

Results
-------

![Celebrate](images/fireworks.jpg){ #id .stretch }

Stream on! {data-background="images/stream.jpg" data-background-color="white"}
==========

---
# reveal.js settings
theme: night
transition: concave
backgroundTransition: zoom
center: true
history: true
css: custom.css
...
