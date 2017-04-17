# Servo iframe navigation

## General Case
The iframe must keep track of the current `PipelineId` of the document inside the iframe. This is
used to get the content window or document, among other things. When a navigation is triggered, a
new `PipelineId` is generated and set as the iframe's current `PipelineId`. This is incorrect
behavior because the associated `Pipeline` does not exist for the newly created `PipelineId` which
will also result in `contentWindow` and `contentDocument` returning `null` until the new document
matures. The iframe should instead keep the old `PipelineId` as the current document until the new
document has matured.

The solution seems simple: wait until the constellation sends a message that the `Pipeline` is now
the active `Pipeline` in its `Frame`; however, the iframe implementation relies on the current
`PipelineId` being set instantly when a navigation is triggered. The iframe uses the current
`PipelineId` to determine whether or not a `load` event should be fired. If the iframe received a
message to fire a `load` event, it makes sure that the `PipelineId` from the message matches the
current `PipelineId`, if it does not, the `load` event is not fired.

### Approach #1
One simple approach is to have the method that handles firing the `load` event also update the
current `PipelineId`; however, this will no longer prevent cancelling/delaying `load` events due to
a newly triggered navigation.

### Approach #2
Another idea is to store two `PipelineId`s on an iframe. One for the current document and another
for the pending load. When firing the `load` event, the iframe would look at the pending
`PipelineId` and check if the `PipelineId` from the message is the same. If so, the event is fired,
otherwise it is delayed. The problem with this approach occurs when the document inside the iframe
navigates itself. The iframe is unable to know what the pending `PipelineId` should be so when the
load event message is sent to the iframe, the `PipelineId` from the message will not match the
pending `PipelineId`.

## Handling `about:blank`
There exist two kinds of `about:blank` documents:
 1. Initial `about:blank`. This document is created when a nesting browsing context is created (i.e.
 whenever a new iframe is bound to the tree). The loading of this document must be done
 synchronously The initial `about:blank` is important because it ensures an iframe never has a null
 content document. During iframe navigation, if the current entry is the initial `about:blank`
 document, the navigation will be executed with replacement enabled.
 2. Non-initial `about:blank`. This document is created whenever a frame naviagtes to `about:blank`.
 Unlike the initial `about:blank`, this load is done asynchronously like any other normal load. The
 difference is that this document must share the origin and script thread of its creator browsing
 context.

Load events with regard to `about:blank` are tricky. Other browsers have different implementations.
There is an open [WHATWG issue](https://github.com/whatwg/html/issues/490) about the proper times to
fire load events.

Hsivonen, from the above issue, proposes the following:
 * Avoid making some `load` events synchronous and other asynchronous.
 * Don't replace one `about:blank` with another.
 * Don't depend on a cache or other race conditions.

> If I had the time, I'd pursue generating the initial about:blank synchronously and queuing a task
to fire events on it. When the task is ready to run, I'd check if the browsing context has started
navigating away from the about:blank document and refrain from firing the events if so. (This is
remarkably hard to do in Gecko, which is why I didn't get this done years ago.)