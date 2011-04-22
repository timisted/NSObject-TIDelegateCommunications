#NSObject+TIDelegateCommunications
*A category on NSObject to facilitate easy communication with a delegate*  

Tim Isted  
[http://www.timisted.net](http://www.timisted.net)  
Twitter: @[timisted](http://twitter.com/timisted)

##License
NSObject+TIDelegateCommunications is offered under the **MIT** license.

##Summary
`NSObject+TIDelegateCommunications` is a category on `NSObject` to make it super easy to communicate with a delegate object.

Given a delegate protocol:

    @protocol StuffFetcherDelegate <NSObject>
    
    @optional
    - (BOOL)stuffFetcherShouldFetchStuff:(TIStuffFetcher *)aStuffFetcher;
    - (void)stuffFetcherStartedFetchingStuff:(TIStuffFetcher *)aStuffFetcher;
    - (void)stuffFetcher:(TIStuffFetcher *)aStuffFetcher didFetchStuff:(id)stuff;
    - (void)stuffFetcher:(TIStuffFetcher *)aStuffFetcher failedToFetchStuffWithIdentifier:(NSString *)anIdentifier error:(NSError *)error;
    
    @end

The category lets you use this code:

    id someStuff = blah;
    [self ti_alertDelegateWithSelector:@selector(stuffFetcher:didFetchStuff:), someStuff];
    
or

    [self ti_alertDelegateWithSelector:@selector(stuffFetcher:failedToFetchStuffWithIdentifier:error:), @"Identifier", anError];

without worrying about whether the delegate does or does not implement the method.

##Important
Note that the category assumes the first argument is *always* the sender so **you should** ***not*** **provide that argument**. 

If you need to call a delegate selector with the only argument being the object that sent it, use code like this:

    [self ti_alertDelegateWithSelector:@selector(stuffFetcherStartedFetchingStuff:)];
    
The category makes use of invocations, so you can use it with selectors involving as many arguments as you like; the only requirement is that **every argument is an object**.

The code checks to make sure that the delegate responds to a selector; if it does, it creates an invocation and invokes it, otherwise it returns immediately, meaning you **don't** need to bracket your delegate calls in:

    if( [[self delegate] respondsToSelector:@selector(dontNeedToDoThis:)] ) {
        [[self delegate] dontNeedToDoThis:this];
    }

Because the code uses invocations and involves `NSMethodSignature`'s `numberOfArguments` method, you don't need to `nil`-terminate your arguments.

##Return Values
If you need to ask a delegate for a `BOOL`, you can do this:

    BOOL returnValue = [self ti_boolFromDelegateWithSelector:@selector(stuffFetcherShouldFetchStuff:)];
    
If a method is optional, and you need the default behavior to be as if the method had returned `YES`, use the *optimistic* version:

    BOOL returnValue = [self ti_optimisticBoolFromDelegateWithSelector:@selector(stuffFetcherShouldFetchStuff:)];
    
Similarly, if you need an object from a delegate:

    id someObject = [self ti_objectFromDelegateWithSelector:@selector(stuffFetcherNecessaryObject:)];
    
##Main Thread
To alert a delegate on the main thread, use this method:

    - (void)ti_alertDelegateOnMainThreadWithSelector:(SEL)aSelector waitUntilDone:(BOOL)shouldWait, ...;

The variable arguments are the arguments to supply on the invocation (again, these should be objects).