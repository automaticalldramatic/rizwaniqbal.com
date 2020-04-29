---
title: "Paginating Firestore collections with snapshot listeners"
date: 2020-04-28T19:05:15
draft: false
toc: true
images: 
  - /images/sky-space-telescope-universe-41951.jpg
tags: 
  - google cloud
  - gcloud
  - firestore
  - pagination
  - typescript
---

There are times when you read the documentation for the way something works, you understand the nitty gritties of it and want to then implement the functionality. However, when you get down to doing it, you find you have questions that don't really answer your scenarios; you browse forums, boards and come out thinking maybe you don't really understand. 

I found myself in a similar situation when I was looking at paginating queries on Firestore while listening to changes from the snapshot.

# Context

My requirement was to be able to display a list of documents from a collection that were sorted by a key and to display changes when a new document is added and display it on the screen respecting the sorting order. The sorting order is based on `priority`. So in my case, when a new document is added, it should always show based on `priority` it was created with. Also, to note, priority is set on a multitude of factors and is seldom created sequentially.

The use case gets complicated later in the article when we look at the way Firestore snapshots trigger the change.


# Code

Lets look at the basic setup. The solution is similar for almost all platforms supported, however, to keep things simple, I will be demoing code in Typescript for this example and using the Web part of the documentation from Firestore.

I will skip the part to [setup Firestore](https://firebase.google.com/docs/firestore/quickstart) for your application and to create a new database instance.

```typescript
// db is the new instance of firestore as per your settings
const db = firebase.firestore();
const cardsCollRef = db.collection('cards');
```

Lets add some data to this collection.

```javascript
cardsCollRef.doc('redCard').set({
    id:'redCard', name: 'Red Card', state: 'Active', priority: 1});
cardsCollRef.doc('yellowCard').set({
    id:'yellowCard', name: 'Yellow Card', state: 'Playable', priority: 2});
```

Now, I create a query to get cards with a limit on the page size and to order by `priority` `desc`, so that the yellow card would be returned before the red card. I also set up a new subject so that the array returned is sent over an observable to be subscribed to by the caller.

```typescript

export interface ObservablePaginatedResult {
	cards: Observable<Array<SomeCards>>;
	stop: () => void;
}

// returns an observable to subscribe to for changes and a method to stop listening to the snapshot
public SyncCards(pageSize: number = this.DefaultPageSize): <ObservablePaginatedResult> {

    // an internal subject to handle streaming until the connection is closed. The buffer size of 1 fits my scenario and is up to the readers discretion
    const cards$: ReplaySubject<Array<SomeCards>> = new ReplaySubject<SomeCards[]>(1);

    // create a query for Firestore
    const query = this.cardsCollRef.orderBy('priority', 'desc').limit(pageSize);

    // create a snapshot query to get cards in descending priority
    const stopListening = query.onSnapshot((cardSnapshot) => {
        const cards: SomeCards[] = [];
        cardSnapshot.forEach((cardDoc) => {
            cards.push(cardDoc.data() as SomeCards);
        });
        cards$.next(cards);
    });

    return {cards: cards$.pipe(distinctUntilChanged()), stop: stopListening};
}
```

If I want to paginate this query and load pages in succession, I need a way for the calling method to ask this service (so to speak) for the next page of data. I also need to pass the new callback returned by the `onSnapshot` method to the caller to be able to stop listening for changes. I chose to use generators for this purpose. One could keep it simple and resort to Promises, as well. I chose generators cause it sparked joy :)

```typescript

// I updated the interface
export interface ObservablePaginatedResult {
	cards: Observable<Array<SomeCards>>;
    nextPage: IterableIterator<() => void>; // the method returned by iterator should be called to stop listening for changes on this page
}

public WatchPaginatedCards(pageSize: number = this.DefaultPageSize): <ObservablePaginatedQuery> {

    
    const cards$: ReplaySubject<Array<SomeCards>> = new ReplaySubject<SomeCards[]>(1);

    const next = this.paginator(cards$, pageSize);

    return {cards: cards$.pipe(distinctUntilChanged()), nextPage: next};
}

	
private* paginator(cardsQueue: Subject<Array<SomeCards>>, pageSize: number): IterableIterator<() => void> {

    // create a query for Firestore
    let query = this.cardsCollRef.orderBy('priority', 'desc').limit(pageSize);
    let lastVisible = null;

    while (lastVisible !== undefined) {

        // create a snapshot query to get cards in descending priority
        const stopListening = query.onSnapshot((documentSnapshots) => {
            // Get the last visible document
            lastVisible = documentSnapshots.docs[documentSnapshots.docs.length - 1];
            if (lastVisible === undefined) {
                return;
            }

            // Update the query variable with a new query that starts after the last visible in the previous query
            query = this.cardsCollRef
                .orderBy('priority', 'desc').limit(pageSize)
                .startAfter(lastVisible);

            // use the same logic as SyncCards to push a card array into the subject
            const cards: SomeCards[] = [];
            cardSnapshot.forEach((cardDoc) => {
                cards.push(cardDoc.data() as SomeCards);
            });
            cardsQueue.next(cards);
        });

        yield stopListening;
    }
}

```

What is happening in the snippet above is that the method `WatchPaginatedCards` creates a generator that takes a page size, creates a snapshot of the documents, updates the cursor and in the callback pushes cards into a subject. [Generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) are functions that can be exited and later re-entered. Their context (variable bindings) will be saved across re-entrances.

This pattern allows us to literally pause a method mid way and resume it later from the state we left it in.

This is how the methods in the above snippet can be used.

```typescript

private snapshotWatchers: Array<() => void> = [];
private syncRequest: Partial<ObservablePaginatedQuery>;

private syncCards() {
    
    // assume some code here to initialise variables, create a service instance et al

    // syncRequest is a class variable
    this.syncRequest = WatchPaginatedCards();
    this.syncRequest.cards.pipe().subscribe(
        t => {
            // do something with the cards received
        }
    );
}

// to get next page
public getNextPage() {
    const snap = this.syncRequest.nextPage.next();
    return snap.value !== undefined && this.snapshotWatchers.push(snap.value);
}

// to stop listening to changes to a particular page
this.snapshotWatcher[3]();

//or all
() => {
    for (const snapshot of this.snapshotWatchers) {
        snapshot();
    }
}
```

You call the method `WatchPaginatedCards()` which creates a generator and returns the instance back. The calling method could now call next until the generator finds the last value. If it does, the generator returns with `done` set to true, which signifies the end of all values and the calling methods could change the state of the application.


# Caveat

As per [the documentation](https://firebase.google.com/docs/firestore/query-data/listen#listen_to_multiple_documents_in_a_collection) Firestore will call a callback method everytime the data in the snapshot changes.  So, when I add a new document, the snapshot for that page changes, however, the other pages do not. 

Example: I am listening to 10 items on every page and have 2 pages in view (20 items). If I add an item that is supposed to be displayed first, my snapshot callback for the first page would send me an array of 10 cards. I cannot simply append them cause they have items that are already displayed (the other 9 items from page 1) and is also missing an item that should now be a part of the second page, but that would mean I refresh the whole query.

 This creates a problem when you have a large number of items displayed on the page and you need to update the priority of a single item and refresh the whole colleciton. 

My solution for this was to maintain a unique [`Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) on the client sorted by priority. 

This way, every time a new document is added, the client takes care of sorting it and displaying it.

```typescript

// this method accepts the property to order by. In our example, this would be priority
// sortByDesc('priority')
sortByDesc(prop: string) {
    return this.returnedCardsArr.sort((a, b) => a[prop] < b[prop] ? 1 : a[prop] === b[prop] ? 0 : -1);
}

// also a slightly verbose way of keeping only unique IDs replacing the ones received from the new call - cause the data also might have changed.
for (const card of c) {
    if (!this.uniqueMap.has(card.id)) {
        this.uniqueMap.set(card.id, true);
        this.returnedCardsArr.push(card);
        continue;
    }
    this.returnedCardsArr = this.returnedCardsArr.filter((foundCard) => foundCard.id !== card.id);
    this.returnedCardsArr.push(card);
}

```

I am hoping this helps. Also, would be glad to hear suggestions. The last part with the for-loop I am running inside a subscribe callback. Would love to hear how one could do this with pipe and it would be more performant.