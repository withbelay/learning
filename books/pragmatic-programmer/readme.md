# Future Topics:


* [https://hexdocs.pm/elixir/1.15.4/library-guidelines.html#avoid-application-configuration](https://hexdocs.pm/elixir/1.15.4/library-guidelines.html#avoid-application-configuration)
* Damn… that link Lee sent out about hexagonal architecture Maybe lee has it. It’s in slack somewhere
*


# Topic 8: The Essence of Good Design


## Examples:



* Jeff
    * Messages to provide isolation between contexts
    * Direct calls across context isolated to the top level
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 9: DRY - The Evils of Duplication


## Examples:



* Jeff
    * The data macros we’re working on are all about finding a single way to express knowledge.
    * The Messages that reuse many of the same fields as other structs. The Message itself conveys something unique about what happened in our system. So it does not violate DRY. And because of this, we need some way of simply including what data it requires. But the second we go beyond this to processing that data the same way in each struct, it is a violation. DefNew is a beautiful strategy for reducing that. DefaultMessageFields and DefaultPolicyFields are too.
    * But OTLP and OpenAPI specs are both in the process of getting fixed here now too. An
    * Notes: CLEAR Code is better than documentation. Documentation violates DRY if all it does is explain what’s happening. It adds value when it explains why and provides context to the business decisions
    *
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 10: Orthogonality


## Examples:



* Jeff

The observers in workflow are ways to keep the steps of the workflow focused on just the stated goal of purchasing a policy, while still being able to fire alerts to investors, open telemetry, recording checkpoint, and console debugging.

Not one of those things would come up in a conversation with Aashay about policy purchases.

Rabbitrouter is another. The messages represent an event in the system. There job is to convey everything about that event. They don’t care who reads it or how it gets there.  Rabbit router provides a single line that allows this routing to take place.  If we changed messaging products, the message struct should not care aside from through this one line declarative interface connection

Areas of concern:



* On the other hand, we have the algoseek code.  Many files all have one purpose. What’s the price of this stock/option. A change in requirements for this can impact any of these files, and you have to go through all of them to figure out which it is. And their apis represent more contracts to maintain.
    * “If I dramatically change the
    * requirements behind a particular function, how many modules are
    * affected? In an orthogonal system, the answer should be
    * “one.’”
    *
    * Excerpt From
    * The Pragmatic Programmer
    * Dave Thomas, Andy Hunt
* Libraries should not force you into doing something special to use it, it is not orthogonal.
    * SSE
        * required us to get data through sse. That’s fine, so long as it’s abstracted away. But it isn’t right now.
    * OTLP
        * This library needs to use decoration, because telemetry applies to every system at belay. If that is the case, then it’s an orthogonal component. Just like it’s useful to IO.inspect everywhere, but if you have too many of them, you lose the thread of your code. Ideally, it should ONLY be used via decorators.
* Kevin
    * Most examples of low orthogonality I could immediately think of seemed to be necessary evils in our codebase. Some that come to mind are Auth0 and the guardian plugs, our configs, backtest
    * For the most part, our umbrella app changes and the fact we were strict with our context separation, led to an immediate high orthogonality.
    * If a piece of code has high orthogonality, the unit testing is usually easy to set up. That was something I noticed that I think is a great test of how orthogonal your module/function is. Usually my process is to hack together some logic that I think is close to how it should work, then write tests to get it running closer to how I want it to. Then start improving the code and the test; this is the part where I can start reducing the coupling my code may have, usually leading to the testing getting simpler. Usually it’s here I can see where the problem lies as I put together the test setup. Usually if I can mock out the lower “layer(s)” of the system, it’s a good sign it’s orthogonal.
* Gray
    * Feels like orthogonality is inversely related to coupling, high orthogonality = low coupling
    * Real examples:
        * I remember our first foray into auth0 as being the first thing we directly called ‘orthogonal’
        * Plugs are orthogonal because they are independent of the controllers and liveviews
        * Even the UI is orthogonal from views, you can change how a site is laid out without impacting the business logic in the controller
        * Elixir config - our different envs have different components that are orthogonal and can be swapped out easily by changing the config, this could be live vs historical markets or it could be entire databases
        * Microservices
    * “What are the differences in orthogonality between object-oriented and functional languages? Are these differences inherent in the languages themselves, or just in the way people use them?”
        * Definitely a how you use it, feels easy to say functional languages are more orthogonal because you can localize changes and reuse functions but the same is true for objects. It feels like it comes down to the public interface, if you change how you use something, that has the biggest impact on whoever inherits the object or function
        * In rust, a struct’s data representation and the functions that defines it’s behavior (traits) are completely independent of each other, and you can even have multiple traits per struct.
            * But you can still totally fuck it up, there’s no rule for what language benefits orthogonality (okay maybe react redux is pretty bad, but its a toolkit technically), comes down to how you design around it
* Lee
    * Test suites are a great “canary” for exposing Orthogonality. If making a minor production code change requires a relatively large number of changes to unrelated test suites, (fiddling with helicopter control inputs).
    * Non-Orthogonal systems examine inputs to “guess” what needs to be done based on the situation they’re applied to or the parameters they’ve been given. While this means that there are fewer functions, their testing is “infinitely” complex and changing them can affect the whole application
    * Don’t defer to runtime that which can be determined at compile time.


## Outcomes:


# Topic 11: Reversibility


## Examples:



* Jeff

Can we change horses from any particular implementation w/out the change impacting the rest of our system? How isolated are our components, and how effectively have we abstracted them? If the goal is to allow our code to handle as many future scenarios as possible, each component should be self contained, with as small an external surface area as is possible.



* Kevin
    * Behaviors in Eliixr are something I’ve grown to like more and more and are great tools in reversibility. At first I didn’t see much use, but slowly started to understand their need with building reversibile systems like our Markets and Messages
    * Some of our least reversibile elements of our codebase, that come to mind, are some of our clickhouse oriented code, if we planned on using that code more across the app (or if we simply have more time), we could improve it. But it’s a cost that wasn’t necessary
* Gray
    * Reversibility: There are no final choices, so you should be able to walk back your changes
        * Personally, I have crossed paths with this one lots of times,I have plenty of personal projects from way back that eventually hit a critical mass of poorly-implemented choices that adds major friction to changes. Never really thought about being able to reverse changes as a feature, but I really like the concept.
    * Examples:
        * We started with a web app, then pivoted to B2B without major losses in dev time.
        * Wiring
            * It’s orthogonal though and has many properties that are ETC
    * Ask yourself “how frustrating would this be to revert?”
        * Orthogonal things are easy to revert
        * Reversible things are ETC
    * Good quotes:
        * “How can you plan for this kind of architectural volatility? You can’t. What you can do is make it easy to change.”
        * “How many possible futures can your code support? Which ones are more likely? How hard will it be to support them when the time comes?” Good questions to ask yourself
* Lee
    * You guys did a great job with the Messages.  By hiding Phoenix PubSub behind the Messages module, it made it relatively easy for me to achieve parity using Rabbit.
    * By putting external sources behind a Proxy of some kind, it not only makes it easier to test, but Reverse the implementation if needed
    * We could even Reverse Ecto if we needed (God forbid) since most of our database access is behind the Context modules


## Outcomes:


# Topic 12: Tracer Bullets


## Examples:



* Jeff
* This one I think is basically the same as Lee’s Driving a Stake idea.
* What’s the difference between:
    * this when it works,
    * this when it doesn’t,
    * and a spike?
* Can you do this in production, or is this a pre-prod thing only?
        * Integration branch for an “entire feature”?
        * Feature flags?
*
* Kevin
    * Definitely like this approach a lot, in most projects, you don’t know the full picture, using the tracer development ideology is the best way to get done what you know needs to be done and get the silos of code integrated together.
    * Another reason why I’m a fan of this approach is because when you expand the code’s features from a good functional starting point means it’s easy if something went wrong with the new iterative features. The author mentions this advantage as: _“Developers build a structure to work in”_
    * I think as a team, we are performing tracer development on a macro application/major functionality standpoint. But when it comes down to narrower focuses, we tend to code differently.
    * Maybe we should take advantage of one of the advantages of tracer development which we don’t already which is: _“Users get to see something working early”_
    * _“You have an integration platform”_;_ _I don’t think we have been taking advantage of this in a tracer fashion. As we added more functionality, we have not been building integration tests to maintain the new functionality, which has led to major breaks. We also don’t seem to be running the app all that often, so we don’t see the breaks for a while. An example of this was our OAuth code was broken
* Gray
    * Spikes: Concept behind the tracer bullet is that you quickly refine code to meet a target you might not have the highest visibility of
        * As you refine the approach, you need to make sure your tracer bullets or spikes are reversible, ETC
        * Help get visibility to the unknown
        * Should tracer bullets be reversible or not? Chapter can’t seem to decide, “Tracer code is not disposable” but also “hello world” is a tracer bullet and “Tracer bullets show what you’re hitting. This may not always be the target.” so do we keep it all or do we slowly refine? Seems to me like the latter.
    * I definitely feel like we did this with rabbit, our first attempt at rabbit wasn’t completely ideal but we refined what we had to get it more on target
    *
* Lee
    * Yeah.  I like this a lot.  I’ve always called it top-down, iterative approach.  Not a spike or throw away (as they mention), but real code that narrows in on the target.
    * My go-to … especially for stuff that’s ambiguous or high risk.
    * Another benefit they didn’t mention is that it allows for smaller PRs; complete from an “aiming” standpoint, but incomplete from a feature standpoint.
    * Also has a benefit that wasn’t highlighted in that, it allows parallelization of the feature.  (The analogy breaks down a little).
    * The only drawback I’ve encountered is that, because it doesn’t actually “do anything” initially, some folks don’t want the code in the repo … which defeats the purpose.


## Outcomes:


# Topic 13: Prototypes and Post-it Notes


## Examples:



* Jeff
    * This is what I did w/ the Partners PoC.
        * [https://github.com/withbelay/belay-api/pull/333](https://github.com/withbelay/belay-api/pull/333)
    * Is this another word for Spike?
* Kevin
    * Seems we mostly do this for architecture
    *
* Gray
    * Something I thought about that the chapter doesn’t mention is that you can prototype on ‘levels’ that match your understanding. For instance balsamiq and figma are two different prototyping tools that offer different levels of detail. Balsamiq is a very post-it note solution where you can drag around premade ui elements to get a feel for a particular design, but figma lets you really tinker with the fine details in what is basically a visual CSS editor.
    * The note about your workplace’s culture affecting whether or not you choose a tracer vs a prototype was pretty interesting
    * Use prototypes to highlight potential pain points, once you start writing a tracer it’s too late. Prototypes are higher level than tracer bullets. Tracer bullets are useful when you know how your application is put together, but not when you are still trying to answer basic questions about your application.
* Lee
    * The nice thing about tools other than code is that upper management can’t get “confused” about it being deployable.  Also, when prototyping in code there can be a tendency to avoid the “duct tape” and still take the time to write good code.
    * Different from POC


## Outcomes:


# Topic 14: Domain Languages


## Examples:



* Jeff
    * DataFrames?
    * typed_schema
* Kevin
* Gray
    * Lots of examples from our code
        * Terraform, dockerfiles, ExUnit, you could even argue that CI and actions are a DSL
        * OpenAPISpex operations and schemas
    * We use macros to create internal domain languages for our various needs
        * Typedstruct + plugins - Typed and validated structs that can be easily inserted into the DB or serialized to JSON to be sent over an API
    * “Give them code that runs, however, and they can play with it. That’s where their real needs will surface.”
* Lee
    * Our macros.  Big win


## Outcomes:


# Topic 15: Estimating


## Examples:



* Jeff
    * Math word problems - Such a crazy super power that most people don’t have
    * Iterative estimation - lol. Come on guys. Welcome to business
* Kevin
    * I tend to always add x amount of time just to factor for the unknown, not really considering what the unknowns could be. This chapter seems to allude to needing to figure out the unknowns to develop an understanding of if something happens, it will then take x more time, and so on.
    * Iterative programming FTW
* Gray
    * It’s hard to estimate things
        * Estimating complexity is easier like we do for sprint planning, but even then we disagree and have different opinions
        * Skill is a large factor in this, if you know how to build something you are better at estimating how long it’ll take to build similar things in the future
        * “The Scotty Factor” but to account for uncertainty
            * Find that when I’m asked for an estimate I usually give what I think it would take, then ~150% of that as an ‘uncertainty factor’
    * Make better estimates by approximating your system: Models > Components > Parameters
        * “Every PERT task has an optimistic, a most likely, and a pessimistic estimate.”
        * Feel like coming up with a good estimate is enough effort that you might as well just omit the best and worst case scenario
* Lee
    * One bite at a time
    * “This needn’t be a paradox if you practice incremental development, repeating the following steps  with very thin slices of functionality” … now where have I heard that before?
    * My wife HATES it when I do variant suggestions.  ““You asked for an

        estimate to do X. However, it looks like Y, a variant of X, could


        be done in about half the time, and you lose only one feature.”



## Outcomes:


# Topic 16: The Power of Plain Text


## Examples:



* Jeff
    * I have to disagree w/ about half of this one.
        * Obsolescence of editors is not a big concern, relative to the life of a typical project. Maybe of a particular SaaS platform, but most binary formats are more stable than they would have you believe
        * Images can convey more information than text
        * The unix command line isn’t as great as people make it out to be. It’s survived because the terminal was line-oriented, and it was the only thing that could work. It was a constraint of the system, not some brilliant design decision
        * Name a CLI tool written using those tools. I can’t think of one. They all use real programming languages these days like go or rust
        * HTML is a great example of something that can be REALLY hard to read if done poorly. It mixes presentation and information
        * Many binary formats have readable segments at the beginning that say how to read them programmatically.
        * A binary format does NOT divorce data frtom it’s meaning if you reject their argument that the “reader” will be obsolete. JPG’s will be around long after our code I suspect. Postresql will still be able to read our databases, even if they have to be updated, and the value of the image over the description of an image seems obvious. Same w/ the value of a powerful database over a folder of text files.
    * All that said, they had some good stuff here
        * Human readable is certainly a positive.
        * Self-describing data (their example where the data was a SSN, and they surrounded it with a label saying what it was)
        * Markdown - It’s not semantic, but it conveys hierarchy and importance
* Kevin
    * Maybe this is an example of lack of experience, but a plain text files within our source is the last place I expect to find actual knowledge. There is only one file I would look at which would be the README file, anything else I might assume is just an artifact of the code unless clearly named.
    * Any other forms of knowledge share I’d expect in external documentation stored somewhere outside the version control or in a tool like Slab
* Gray
* Lee
    * I agree with Jeff with most of his sentiments.
    * There does seem to be a subtle undertone here though in that I don’t see them saying “nothing but text” but (like so much of the book) “look to text first for its simplicity, ubiquity, and longevity (though longevity is a poor excuse to me)”
    * To me, one of the major reasons that Unix has done as well as it has is due to its “toolbox” of simple, focused, “chainable” tools.  As opposed to many environments where many things are done by many tools where each has a different flavor of common needs such as searching and filtering.  The tiny focused tools are easier to understand, reuseable, and aid development because of it.
    * I really do wish that the industry would adopt a means to have images embedded, but that’s probably not feasible due to their size.  Better to make the code itself more descriptive.
    * XML is descriptive, but would never willingly adopt it.


## Outcomes:


# Topic 17: Shell Games


## Examples:



* Jeff
* Kevin
* Gray
* Lee
    * I see the value of what they’re saying, but I don’t find it as valuable as the other guides.  My brain is limited, and most investments I make are into how to write better code faster.
    * The weakness of shell extensions is that I have to find them and spend time figuring them out. … using up valuable mental real estate
    * Instead, I extend IntelliJ or add an IDE run command.  This makes the command easy to find, I can assign a keybind if I do it a lot, which can even have its own history.
    * Can’t easily share with you jokers, but that’s a small price


## Outcomes:


# Topic 18: Power Editing


## Examples:



* Jeff
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 19: Version Control


## Examples:



* Jeff
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 20: Debugging


## Examples:



* Jeff
    * Bisect can be useful
    * Clear tests are good
* Kevin
    * +1 on before looking at the bug, make sure that you are working on code that is built cleanly. I wish we can get back to no warnings, and keep it there
    * Write a test for bug that was found, both for reproducing while debugging and for longevity of the fix
* Gray
    * “Fix the Problem, Not the Blame”
        * Feel like we’re really good about not pointing fingers at each other over bugs, and we’ve always had this culture for as long as I’ve been working here
    * “Before you start to look at the bug, make sure that you are working on code that built cleanly—without warnings”
        * You heard the book, start cleaning up those warnings
    * ​​Rubber Ducking/Process of Elimination
        * Sometimes the problem is between the chair and the keyboard, which is why rubber ducking is effective
        * But sometimes there are legitimate problems with the libraries we choose
            * Clickhouse elixir libs were unmaintained
* Lee


## Outcomes:


# Topic 21: Text Manipulation


## Examples:



* Jeff
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 22: Engineering Daybooks


## Examples:



* Jeff
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 23: Design by Contract


## Examples:



* Jeff
    * You start ahead of the curve using a functional language, because state is harder to get access to, But of course state is needed at times.
    * I struggle with where and how much to use Wiring, because the state isn’t passed in to the function. That can hide what is going on and couples that code to that state’s behaviour.
        * Usually, I would make that trade if what’s happening feels orthogonal to the primary intent of the function. So wiring initializes a server w/ regard to knowing it’s place in the world. But if it was being used for what the server’s primary objective is, I’d be more concerned.
    * Embedding the contract itself in code
        * OpenAPISpex - we’re establishing 1 half of the contract, and trying to enforce it programmatically.  Then, in generating those API docs, we’re trying to allow code to be autogenerated, so that the contract is programmatically encoded on both ends
        * Messages context is also design by contract. Just indirectly. The messages themselves are code, and are versioned. And their intent is implicit in their name. And they represent a contract as well. Just a 1-way contract. Even our routing macro is a declarative contract (it tests itself against the fields of a struct at compile time), and you subscribe to that contract by sending an instance of one in when subscribing and publishing.
    * Question: Can this principle be used to tell when you’re doing too much in a function? They talk about being lazy in what you will accept, and lazy in terms of what you will offer. But I don’t know that there’s an easy way to feel whether you’ve got that right or not
    * How do we feel about how they put DbC as something that offered different benefits to TDD? Like you’d use one for some things, and one for others.
    * Could our red spam with Offerings and Markets be a DbC error?
    * We tend to not pass bare maps and keyword configs around much
* Kevin
    * Sounds like the chapter was just reiterating then main design patterns of Elixir and what elixir offers us
* Gray
    * We are designing with contracts
        * APIs are a contract
            * Open API Spec enforces the contract with assertions on operations
        * Messages are a contract between our services
        * Specs help define contracts, pattern matching helps validate
        * Like the book says, elixir is really good at this and we do it without thinking about it
    * Crashing early
        * Something we need to do more of, if something goes wrong we gotta blow up so we know what’s going wrong
        * Ran into this issue with partner config being permissive until the workflow was actually being ran
    * Dynamic contracts & agents
        * Sounds like a pain to test
        * Is versioning easier or harder with dynamic contracts?
* Lee
    * Pattern matching on properly validated structs is a contract
    * “One function to rule them all” is a Design by Contract anti-pattern


## Outcomes:


# Topic 24: Dead Programs Tell No Lies


## Examples:



* Jeff
    * Interesting that they equate catch/release exceptions as a form of coupling. I guess I see their point, but it seems to be that if you’re calling a function, anything it could do is gonna couple you to it whether you know it or not.
    * Typed Parameters is a form of defensive coding I think
* Kevin
    * Most places where we fail this concept have always been hard/annoying to debug. Failing fast is the way
    * Envs are a great place to fail fast, eg: System.fetch_env > System.get_env
* Gray
    * I feel like generally we’re good about this principle
        * Partner config had this issue (would fail too late and wouldn’t properly raise the error)
        * Easy to get caught up with handling errors in with blocks
            * Powerful when you know you want to handle an error in a certain way (payments failing for instance)
            * But equally powerful when you let it blow up
    * Are dead letter queues violations of this principle - if you come across a message you can’t handle, your app doesn’t blow up, the DLQ requeues the message and tries again. Seems like they embody a “don’t crash, don’t trash” concept
    * Where’s the line drawn between fault-tolerance and crashing early?
    * “A dead program normally does a lot less damage than a crippled one.”
* Lee
    * This is the key reason I do not like Go
    * Data validation at boundary layers is key.  E.g. it’s not defensive programming
    * Even truly perfect bug-free code can fail
    * One thing to remember is that it can take time to recover resources and locks from a dead process so a supervised process can fail on restart because the resources it needs are still held by dead process (but will be available soon)


## Outcomes:


# Topic 25: Assertive Programming


## Examples:



* Jeff
    * Again, our structs help us validate the shape of our data
    * DefNew coupled with changesets validates the content and format of that data
    * Pattern matching against simpler types, like strings can let us create assertions against simple func inputs, and I don’t know, but it’s possible we could use these more
    * It’s possible I’m just stuck in a rut here, but I still think our most pervasive gap at the moment is in our config validations
    * Our macros are making a lot of this work better all the time. They’re baking in these assertions. They are forcing a single path in to execution that can be tested once for all. There were a lot more gaps in our code just a month ago.
* Kevin
    * Pattern matching as specifically as possible is probably the cheapest way to create an assertive program that is not super intrusive to the code flow.
    * Explicitly using assert though does not seem favorable in my opinion. There are a million things your values should not be, but only 1 thing it should be. Just make sure it’s that 1 type
    * If you find you need your code to run assertions to make sure something is not an unexpected value, then you probably need to ask how that unexpected value got in there
* Gray
    * I’ve always used assertions as a testing tool, never thought to just throw them in the code, not sure how I feel about it
        * Feels like a test suite where I can’t specify what tests I want to run
        * Guard clauses work as assertions in code but they don’t really feel like it
    * Seems like the point of the assertions is to test while code is running, but then the chapter states you’ll have to remove some of them in prod, so why not put that effort into a test suite?
    * Is observability a replacement for inline assertions? I like the idea of being able to see issues as they occur, but maybe we can get that for “free” without needing to write assertions everywhere
    * “Use Assertions to Prevent the Impossible” is just as applicable to tests as it is the inline assertions
* Lee
    * I’m not convinced of the value of assertions littered all over the codebase.  They add noise, take time to write, and potentially introduce their own bugs.  I feel DbC, validated data, and “let it fail” are far superior.  I actually can’t remember the last time I did this in production code.
    * Configs are definitely our most reasonable application of assertions at the moment.  Standard Elixir makes this difficult as you’re back to the Application Singleton.  Using Injection makes it easy to test the code, but isn’t “natural” in Elixir.
    * Because Configs are literally different between test, dev, and production, testing is non-trivial.  What works fine in test and Dev can fail horribly in prod.  I honestly don’t see a good way to solve this without a “proxy” (like BelayApplication) that can validate state early … Dead Programs Tell No Lies. Extensive integration tests can do this for us, but are more expensive to write and maintain.


## Outcomes:


# Topic 26: How to Balance Resources


## Examples:



* Jeff
    * Ok, this one I loved, though not because of the specific topic. But they called something out that was really good.



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


* This highlights something I feel like I’ve done a poor job of conveying. Coupling at runtime. In this case, they say 3 files are coupled via that customer_file, and that’s bad. I agree. And I think this is useful when thinking about splitting up modules into smaller modules. It can be hard to argue whether the modules in those files have different responsibilities. But if they rely on the same data being passed between them, then they are coupled. This example is a worse case, because it’s an open resource, but the ideas still applies I think.
* Kevin
    * Not sure how applicable this is to our code at first read
* Gray
    * We don’t do a whole lot of reading from files
    * But we did run into a kind of this error with our first market proxy iteration
        * We’d fill up an ETS with market data until fly ran out of memory and the app restarted
        * Are we still using ETS anywhere though? This was an old solution
    * Is it possible we could run into a similar issue with dataframes?
        * Our hedging data frames get pretty big, they pull in a whole option chain, if we need to hedge a lot of symbols at once, that’s a lot of big dataframes sitting in memory
    * Being able to observe resource allocation is important for solving these problems
* Lee
    * Rabbit has a lot of resources we must be careful with.  (and we’re not yet).  One library I like uses macros to support a “with &lt;resources> do` semantic


## Outcomes:


# Topic 27: Don’t Outrun Your Headlights


## Examples:



* Jeff
    * Building software is NOT driving a car. The analogy is flawed because driving a car is stateless. The decisions you’ve made do not build on one another. There’s nothing foundational. It’s just making decisions as you go.
    * This analogy is valid, but the implication offered that ties our ability to see the future to how ineffective a car’s headlights are at speed gives the reader the implication that as software developers we can never see far into the future either. And I think that that implication is less true. It also implies that headlights are an external thing that can not be changed.
    * It also implies that there’s only 1 path forward. The lane you’re driving in.
    * So all of this paints what I think is an incomplete picture. In my opinion, the situation is more like chess.  How many moves ahead someone can think. And not of one particular strategy, but of all conceivable strategies possible from that point further. More experience can result in the ability to see further and more broadly. I’m not arguing that you should do big design up front. But I am saying there are things you can anticipate needing, and that in all future scenarios the product might take, you would need them. And so it makes sense to build them even if the current point doesn’t instantly require them. My goal is to provide maximum flexibility you can achieve that with a blend of building intelligently for the future (good foundations) and doing the minimum you know you need. Eg: Get headlights that go further and wider
    * The Black Swan, and Nassim Taleb’s work in general, is really great, generally
    * They talk about easily replaceable, and good mechanics being key if you’re going to not look forward to the future too far. Or even if you are. I certainly agree w/ that.
* Kevin
    * I understand the point that the topic is trying to make, however I agree with Jeff that you need to take some calculated risks in planning the future you expect to be at. Doing so with easily “swapp-able” code seems like the way to easily backtrack and swap components
    * Some of the “least changed” code has been code that we avoided doing until the last minute. I’m thinking things like auth, partners (to an extent, everything up until the env config stuff, but that was easily removable)
    * I like the idea of this, but I think we need to break it sometimes. It’s not a rule
* Gray
    * “Consider that the rate of feedback is your speed limit.”
        * We’ve had lots of discussions about our PR review speed, because that is our real speed limit. You can only juggle so many at once
        * Pairing has really helped keep the feedback high
        * Standup reviews have helped keep feedback high
    * Sometimes you need to make a choice based on some uncertainty
        * Tracer bullets and spikes help you orient your headlights
        * ETC - if you go down the wrong path you need to be able to unwind the changes
    * Black swans - It’s easy for us to speculate but it’s impossible to predict the future.
    *
* Lee
    * Small, incremental work whose direction is independently evaluated by other team members before too much has been invested
    * “Fortune telling”, “avoid guessing about future needs”


## Outcomes:


# Topic 28: Decoupling


## Examples:



* Jeff
    * “Developers who are afraid to change code because they aren’t sure what might be affected”
    * I struggle with them saying this code hits 5 levels of abstraction: I agree it should be broken up, but when I think of “Levels”, I think of my airplane analogy where it’s like, zoom levels. To me, these are all separate concerns on the same level of abstraction. Doesn’t change the outcomes they present, which I agree with. But the language here matters for us for future discussions. To go to a deeper level of abstraction, I’d think you’d need to have integrated that discounts can’t be more than 40%, for example.
    *

<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")

    * Pipelines != `|>` - That said, if you’re not using some of the primitive Enum functions, I think it’s a quick path to confusion if the fundamental type of data changes as it goes through the chains. Having a context object is much easier, IMO.
    * “Each piece of global data  acts as if every method in your application suddenly gained an additional parameter:” &lt;- SO well put
    * I think we have a tendency to decouple things too much. And the other day I was reading something that highlighted the idea that you can tell if you’ve pulled code out into too many modules when you consider the shared data between the function calls. The more functions and more data being shared, the more coupled your “decoupled” architecture is. So making sure a module can do its job independently can mean fewer functions to call and parameters to pass
* Kevin
    * “Developers who are afraid to change code because they aren’t sure what might be affected”
    * “Each piece of global data  acts as if every method in your application suddenly gained an additional parameter”
    * I agree that those 2 quotes are great things to remember when examining how coupled our code is. I find the first one especially important.
    * By its nature, our trading code is coupled, since at the top level we need to run through multiple parts of the code. Not sure how that can be addressed.
    * I think our decision to try and make sure all our apps access other apps with as thin of an API as possible helps start the decoupling we desire.
* Gray
    * Pipelines vs Trainwrecks
        * I feel like we’re good about chaining function calls, they don’t trail on for too long and complex functionality is delegated to private functions.
        * Delta hedging I think you could make the case that it was a train wreck when it was all elixir maps, but we were good about moving it to data frames and further simplifying it
        * Context objects are useful for piping, think about setup calls in exunit or liveview sockets
            * But I think it’s possible to make a complete mess of them, especially in a liveview scenario. What happens to your socket when you mount, mount for the second time, handle params, handle a live patch, or get a pubsub? It’s hard to tell what your socket state looks like as the live view complexity grows
            * A typed context object is more work but might be better in some cases, purchase flow state comes to mind
    * Global variables
        * We don’t really have global variables, just config but it creates the same problems
        * Some modules have their own global attributes but I can’t think of any problematic ones off the top of my head.
        * Config isn’t _really_ global either since we configure the various contexts, but the issue with it is that it all has to be there and exists implicitly
            * Having a lot of config is a huge pain, and each time we add a new repo we compound the problem. Jeff’s 1pass/terraform stuff is awesome to automate it but someone has to think about what config is needed at some point, is there a way we could better balance complexity and configuration?
* Lee
    * One form of Coupling people frequently overlook is testing and the same principles apply. Code is not “refactorable” if you have to throw out or radically alter your tests every time you refactor the production code.  Why have tests at all if they’re so fragile/Coupled?
    * Tests are to provide a safety net to catch you when you make a mistake. If, every time you change your production code, you find yourself having to change the tests, the production code, not the test, is to blame.
    * Making internal/private functionality public so it can be tested is a Coupling smell.  Consider extracting that functionality and using Composites, Factory, or Injection to make it available.


## Outcomes:


# Topic 29: Juggling the Real World


## Examples:



* Jeff
* Kevin
* Gray
    * Events represent some work or action that has been done
        * Extremely common in the web layer, vanilla JS lets you create event listeners
        * Phoenix inherits this behavior, instead of adding an event listener in JS, you have event listeners written in elixir (phx-click is just an onClick event, mount is an onLoad callback
    * A GenServer is a FSM that performs the state transitions in response to events
        * We’re utilizing this concept a lot already. A LiveView is a state machine driven by the events from the user and events from the business layer. Policy workflow is a state machine. The design of elixir helps you write the branching paths of state machines with features like pattern matching in the function declaration.
    * How much work should an event encode?
        * “It can even be something as trivial as fetching the next element in a list.” But should it? I feel like this event is trivial unless there’s some external constraint that requires an event like SSE or web sockets
        * It feels like we’ve been creating events that have a minimum ‘size’ constraint, because we don’t have a lot of these trivially small events. I think having a lot of those micro-events doesn’t actually make code more decoupled or ETC, it adds an overhead that makes the system harder to think about as a whole.
        * An example of this - you wouldn’t write a GenServer to parse a string. Even though a GenServer is a FSM, it’s not the right tool for the job. Enum reduce is also a finite state, but you’d use a lot less code and time to write that than a GenServer
* Lee
    * Regarding FSM: I haven’t found much need for them.  Really rare.  I wonder if it’s due to the business domains I’ve been in. And now, with Elixir’s Functional Programming, there seems to be less “day to day” need.  (The book indicates “weekly”).  Not convinced.  Good tool to know of though.  Like the Workflow might have benefited from being a “real” FSM




## Outcomes:


# Topic 30: Transforming Programming


## Examples:



* Jeff
    * In a previous topic, they had code that looked a bit like this, and said it was bad. And yet here it was good. What distinguishes them
    *

<p id="gdcalert3" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image3.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert4">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image3.png "image_tooltip")

    *

<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image4.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image4.png "image_tooltip")

    * This felt like flowery bullshit:
        * `In the transformational model, we turn that on its head. Instead of little pools of data spread all over the system, think of data as a mighty river, a flow. Data becomes a peer to functionality: a pipeline is a sequence of code → data → code → data…. The data is no longer tied to a particular group of functions, as it is in a class definition. Instead it is free to represent the unfolding progress of our application as it transforms its inputs into its outputs. This means that we can greatly reduce coupling: a function can be used (and reused) anywhere its parameters match the output of some other function.`
        *
        *
    *
* Kevin
    * Everything in the chapter reminded me of the concept of a “black-box” and how responsibility needs to be split up. Each transformation is like a black-box and has a distinct goal. Not sure if these are synonymous but it felt like that.
    * With statements are our way of piping while also checking the result
    * As much as I love Elixir and the pipes, I don’t think you need them to perform transformational programming to write good code. Even in elixir, sometimes trying to force some function to work in a pipe just leads to messier code
* Gray
    * “​​If your background is object-oriented programming, then your reflexes demand that you hide data, encapsulating it inside objects. These objects then chatter back and forth, changing each other’s state. This introduces a lot of coupling, and it is a big reason that OO systems can be hard to change.” - We use message objects all the time, but I’d argue they allow us to make changes easier than if we didn’t have them. I think what makes this model work is that the messages are data but are immutable, so handling a message requires a different message to be sent out. It’s a distributed pipeline where you can tap in more handlers where you want without having to make a change in the rest of the code.
    * “It would be nice if Elixir had a version of the pipeline operator |> that knew about the :ok/:error tuples and which short-circuited execution when an error occurred.” - with statements???
    * Code that interacts with dataframes is a good example of these transformations, hedging particularly
    * I think what makes the transformational model work is a common data type, in hedging we have data frames, in unix we have text, in belay we have messages but I think over-relying on a data type for everything just naturally ends up overloading the original intent of the data. With unix utilities specifically there’s a degree of text mangling that occurs from one function to another as you try to coerce your outputs into inputs. Sure it’s powerful, and flexible, but the data suffers. JSON objects are another good example, at a certain point they’re just unreadable
* Lee
    * While their `and_then` seems a poor substitute for a `with`, it’s clever and there might still be some use for it.


## Outcomes:


# Topic 31: Inheritance Tax


## Examples:



* Jeff
* Kevin
* Gray
* Lee


## Outcomes:


# Topic 32: Configuration


## Examples:



* Jeff

    Configuration has a lot of problems:

* It’s hard to know if you’ve configured it correctly before runtime. Eg: That all values make sense, and all the values are there, etc
* The combination of configuration values may not behave well in production

    Generally speaking, less Configuration is better than more. That means sensible defaults that mean fewer knobs to turn result in a system that will cause less trouble as you deploy to different environments. The apple approach of just 1 way this can work, and 1 way this can integrate is a lot more reliable than the linux approach of everything can work if you just get these 1000 variables to match up properly.


    Configuration has a hidden cost too. It’s not code, but you often must read the code to understand what your configuration value is actually doing.


    Fewer changes between environments == less configuration

* Kevin
* Gray
    * How can we keep config to a minimum? As what we build grows in complexity, so does the config. And on top of that we have config for 4 environments, in multiple apps, and GitHub CI needs to be aware of secrets and config as well.
        * Configuration-As-A-Service - First time hearing about this idea but it makes sense, what we’ve done with 1password feels like configuration as a service. But are there other tools that can help manage them? 1pass is kind of just a big list at the end of the day and honestly not any more or less readable than an exs file
        * “configuration should be dynamic, is critical as we move toward highly available applications. The idea that we should have to stop and restart an application to change a single parameter is hopelessly out of touch with modern realities.” Isn’t this how fly works?
        * “Don’t write dodo-code” - goal is to use config to let apps be flexible, but config is inherently brittle.
* Lee
    * Gotta be testable
    * Testing should have reasonable expectation that it’ll work in prod.


## Outcomes:



* More structs for our complex config values
* More tests to ensure that our configuration will work in all environments


# Topic 33: Breaking Temporal Coupling


## Examples:



* Jeff
    * ActivityDiagram, as a layered thing to add to flowcharts, was new to me. Though the only thing I really saw that was new were the teency diamonds. Unfortunately, mermaid does not support either those or the join bars
        * One interesting thing application of these, would be to model policy activation. Because that message goes to trading and insurance, and ultimate ends up sending a message back to policies (unsold inventory)
        * Claims would be similarly interesting.
    * We designed our system to be async almost by default. And we’re very very good about splitting things out using messages. In fact, the change to UnsoldInventory was a good example of us doing exactly this.
        * The downside is that I think this is related to our red-spam discussion from just a few minutes ago. Because 1 message can concurrently trigger several other actions, when we only care about one of them, we can get ourselves into trouble.
        * To resolve this, the processes that are doing the parallel work need to not be spun up, where possible. This may imply more work on supervisors to help encapsulate these related processes, but I’m honestly not sure.
    * Backtest is our hardest animal to deal with here. Because the top level concern was a tick. And we didn’t want the backtest, given that it’s really just a harness for manipulating time, to know how our system works. But the problem is that it’s not easy to know when “done” is. So we’ve done a half-measure, where triggering processes are configured to have completion processes to watch out for. And so we create those “join bars” dynamically based on what’s going on. But that still is a form of coupling, because of our async nature. Hard to know for sure what will ultimately happen as a result of each trigger. If our system were synchronous, this would be easy. But unfortunately, it may be that a timeout mechanism will be required as well. That said, it would help us explore the problem domain to have an activity diagram of it.
    * Here’s a fun thought… Lee’s forthcoming correlation and causation ids, coupled with the centralized nature of our tracing infrastructure, ought to let us create these diagrams automatically for absolutely everything that uses them.
* Kevin
    * I really liked the Activity Diagram, it seems like something we could apply to our main business logic aka policy purchase -> hedging
    * The event based system (I think) helps with keeping everything as concurrent as possible, if we ever plan to increase the nodes, we could have multiple machines doing the work for us
    * I’m curious to see if there are parts of the code that are synchronous now that could benefit from utilizing Eliixr’s Tasks more to create even more concurrency
    * We will be able to generate a Activity Diagram for our major business processes via OTLP once we get our traces in order, showing from start to finish all the spans (steps) that occurred
* Gray
    * Modern languages make concurrent software easier to write (node, go, elixir), I think it’s something programmers are thinking about more
    * Activity Diagrams remind me of sequence diagrams, but now multiple events can happen across time
        * Some of our workflows might benefit from an activity diagram analysis
    * Observability & reporting are key to maintaining a concurrent system, it’s comparatively easier to debug in a sequential system where order of operations is guaranteed, but as the number of events and “joins” increase it becomes harder and harder to effectively track the flow of data - messages - through the systems
    * Curious as to how scaling multiple machines or instances of belay will affect the system. Instead of concurrency “locally” we would have concurrent belay systems running for various regions
* Lee
    * We REALLY need to do this for Backtest
    * Regarding the module dependencies for Elixir compiles: The inter-module dependencies that can be created by macros will cause heartache (I’ve seen it).  It can really slow down the compile, or end up with modules that compile differently based on the “temporal” order of compilation.
    * As our system increases in complexity, this mapping will be more important and harder.  I wonder if it would be possible to automatically analyze where our message structs are produced and consumed. That map could then be examined for places where synchronization is needed.  Again, something that’ll be needed I think for a stable and reliable Backtest.  The OTLP tracing data provides most everything we would need …


## Outcomes:


# Topic 34: Shared State is Incorrect State


## Examples:



* Jeff
    * The problem we hit w/ the database errors because multiple processes were trying to access the same rows was the database doing it’s job w/ regard to locking (semaphore) rows, two blocks of our code trying to mutate the same rows, and only one being allowed to succeed. Databases these days can lock either the entire table for writes, or just specific rows.
        * Optimistic concurrency - We clear on what this is? It’s a little different
    * The semaphore example is just like the with_channel func that Lee merged in last night.
    * When dealing with shared state, these strategies work, but a lot of times, the easiest strategy is to serialize access to the data. In other words, only one process can ever write that data, but anyone can read it. Ets tables have this concept, and the change I made to UnsoldInventory did this as well. The subscription to Inventory changed is now handled by just 1 process. But all of the Offerings processes are free to read from the database whenever they like.
* Kevin
    * Random Failures Are Often Concurrency Issues: After reflecting on past difficult issues we have encountered, I realized that they have been concurrency/race conditions. I’m gonna be keeping that in mind now \
I do not know what optimistic concurrency is
    * The place I think of these problems occuring has been the DB, everything else we have tried to split as much as possible down to partner/sym/policy
* Gray
    * “Random Failures Are Often Concurrency Issues“
    * Can we identify where shared resources will cause mutual exclusion issues before they show up?
        * Seems the easiest way to get alerted to the issues is when sentry blows up, it would be nice if we didn’t have to wait that long
        * Can we identify shared state programmatically?
    * What happens when multiple rabbit exclusive consumers run? The exclusive consumer is kind of a mutex for our message queues, so how does delivery get affected by multiple consumers?
    * Elixir doesn’t really have the concept of a mutex  in the language by default. Seems like they just expect you to sort it out
        * [https://hexdocs.pm/mutex/readme.html](https://hexdocs.pm/mutex/readme.html)
        * Just a singleton genserver that manages access to a resource. Multiple readers, one writer
* Lee
    * Recognizing where and why race conditions will happen is never ending.  It seems that every bit of code I write, I’m asking myself “how can this have concurrency issues”
    * Sometimes the best way to deal with it is simply an apology.  (Optimistic locking).  After all, realistically, how often will two diners want the the last slice of pie at exactly the same time


## Outcomes:


# Topic 35: Actors and Processes


## Examples:



* Jeff
    * Ok, thought this would be more interesting. And to anyone working in an INFERIOR language it would be. But… I guess this is just where we live.
    *

<p id="gdcalert5" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image5.gif). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert6">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image5.gif "image_tooltip")

* Kevin
    * If only we had a way to do this in Elixir, shame…
    * The JS actor model they showed reminded me a bit of redux/flux but less opinionated.
* Gray
    * “When processing a message, an actor can… send messages to other actors that it knows about” They don’t have to know about each other if they use PubSub, they just have to know about the message. This is the problem with the JS implementation, actors are coupled to each other by a reference, what if you wanted to add another waiter? Another pie case?
    * “There’s no single thing that’s in control. Nothing schedules what happens next, or orchestrates the transfer of information from the raw data to the final output.” I think this is what makes backtest so hard. Time (or the market) orchestrates belay - market events prime the system and define when the system can operate and our actors depend on that because they depend on the effects of MarketReady and MarketClosed.
* Lee
    * duh.


## Outcomes:


# Topic 36: Blackboards


## Examples:



* Jeff
    * Familiar notes on messaging that tie into things we’ve done and are doing
* Kevin:
    * Sounds like our contexts are the same as “blackboards”, and we are building the tracing through the messages and actors. Once we get it right, I imagine we will reduce the downside of blackboards
    * I wonder why they only mentioned Kafka and NATS, not rabbit or any other type of simple messaging service. Is persistence important?
    * Maybe it would have been overly complicated, but our purchase workflow seems like it could adopt a blackboard system, which would allow the steps that don’t rely on each other to work asynchronously.
* Gray
    * ETS feels like the system closest to a blackboard, processes can write to it and other processes can read the updated state from it
        * But it feels like the blackboard “accumulates” the final result of the problem being solved by the various processes, and we don’t really use ETS like that
    * Blackboards decouple the solution from the solvers. But a blackboard needs to be smart to be useful:
        * But how do you alert whoever is interested in the solution that it’s ready?
        * How do you alert the consumers that their data is ready?
        * How do you prevent processes from stepping over each other?
            * Imagine one process takes the blackboard state and tries to run some calculation, but in that time another process changes the blackboard state. When the first process finishes, it would probably have to just discard its solution to avoid errors. But now you’ve wasted work.
            * You could use a mutex (but that might impact performance) and if the processes are serial enough you could just use a workflow
* Lee
    * Mentioned that before that I’d like to serialize all messages. I’m not sure yet how useful it would be to us, but it is another piece of data that might be useful debugging.  The updates to Rabbit stuff would make this a lot easier.


## Outcomes:


# Topic 37: Listen to Your Lizard Brain


## Examples:



* Jeff
    * This one happens to me a lot. I’ve never seen it put like this, but there are a lot of times I know something is going to bite me but I can’t tell why. I’ve been working on getting better about being able to articulate the reasons. That’s part of why I picked this book for us to discuss. To give us a shared language, and to make me better at figuring out what it is that I see when I see things.
    * Our rabbit code has always given me that sense of disquiet, and with the channel management it still is. However, I really like our general approach of pub/sub.  I just found the package `rabbit` - It’s REALLY good. But it forces your code down a path that mirrors rabbit’s own architectural decisions. And… ours are better. Starting with pubsub, and that simplicity. And then creating a rabbit client that matches that simplicity is something that I think Lee did a REALLY fantastic job of. Because the encapsulation has just worked, and I’ve never felt restricted by it. But to do it the “rabbit” way is very complicated, relative to how GenServers work. Or how pubsub works. And for what? Nothing that I can see. So I’m excited to get our vision of this to run smoothly
    * Other times w/ our code I’ve felt this:
        * Wiring
            * V1
            * V2
            * V3
            * V4
        * Kafka - when I was still thinking that direction
        * CQRS - when I was still thinking about that
        * Auth0
            * That miasma for months
        * Rabbit
        * Algoseek
        * Backtest
        * Trading
            * Pre-dataframes
        * Purchases
            * Pre Workflow
* Kevin
    * When things feel hard or wrong, taking a break is usually the best thing, the next best thing is talking about it. This is something that I realized was what saved me going down a path that would have bit me in the ass.
    * Generally this chapter felt a bit like a pep-talk
    * My lizard brain tends to flair up whenever we are creating a new feature that is starting off more than 1 layer abstracted. I find that when something is abstracted right away, it can lead to walking into a dead-end that’s a mile deep. I rather start the least abstracted as possible and build the abstraction once everything works but maybe the code isn’t pretty/efficient yet.
    * Another time my lizard brain flares up is when a **big** refactor/reworking PR gets merged since I just have a feeling that there will be something we missed.
    * Times I’ve felt it: Wiring, rabbit, mkt_proxy re-write, auth0 refactor, the umbrella refactor
* Gray
    * I’m a big second guesser when I’m pairing but find that when I’m working alone I’m not as critical
        * Not being as critical -> lizard brain is doing too much coding
    * I don’t really have issues with starting things, I find myself second guessing my changes to existing things the most
        * I don’t want to make the wrong choice and have to redo a PR
        * And I think that’s what it is, I make the assumption that second guessing saves time overall because the less time I spend tweaking a PR, the more time I have to work on the next one
    * I don’t know if the “Playtime!” Experiment/technique would really help me, feels like my concerns about second guessing are a little different
    * “Listen to your instincts and avoid the problem before it jumps out at you.”
* Lee
    * Taking time to wait for the lizard brain can only work if you’ve slogged long enough in the code to give it a chance to load up the details
    * The “I am prototyping” thing is just the top-down, iterative development.  Do the stuff you know needs to be done while letting the LB work on next steps


## Outcomes:


# Topic 38: Programming By Coincidence


## Examples:



* Jeff
    * I feel like this is related to Lee’s desire to understand and remove red spam
    * I think this is also why I want to refactor code that feels like it’s harder than the problem it’s trying to solve justifies.
        * FWIW, this is why I so enjoy our abstraction on top of rabbit. The underlying bits are harder that the idea of: “just get me the messages that look like this”
    * Tests are only as good as the tester writing them
* Kevin
    * I find we avoid any of the easy pitfalls of programming by coincidence just through our testing standards being so high. You can’t write a good test without understanding the code you wrote
    * Pairing on rabbit has felt like programming by coincidence at times since I haven’t had the time to go through the docs
* Gray
    * You can also architect by coincidence. Code might work and you might know what it does but if you don’t know how all the parts come together you might code yourself into a situation where adding features is impossible
        * Ran into this a lot when I was starting out
    * It’s easy to write tests that reinforce coincidences
        * Super easy to in live view tests
    * I like to mention the assumptions I make directly in the PR, it helps me keep track of what I’ve done and also lets other people come in and discuss. I feel like assumptions are the biggest pain points in PRs so making them obvious helps speed things up
* Lee
    * A subtle testing trap … when you don’t code against the public interface, but shortcut and call a “private” function made public, (like call `handle_info` directly) you introduce a gaping coincidental outcome.  You’re relying on expected behavior of other private behaviors of the code under test.  And yes, … it might work this time (probing for mines) but future devs can easily, and unknowingly introduce a bug because they change “private” behavior/state that your test was assuming. The chances for failure go up substantially, the larger the module under test … which will likely mean you have a large test suite … which reduces the likelihood you’ll happen to see the dependency on “private” state/behavior.
    * I think the last statement may be the most important and one that Jeff is really good about. “Don’t be a slave to history. Don’t let existing code dictate future code.”
    * “I’m gonna code me a minivan”  Real life
    * https://devhumor.com/media/dilbert-s-team-writes-a-minivan


## Outcomes:


# Topic 39: Algorithm Speed


## Examples:



* Jeff
    * I find big O analysis to rarely be useful. I’m more often struggling with memory, and too many network hops, and how to parallelize slow code. And db indices
* Kevin
    * Almost all code we have written usually takes O(n), like Enum.maps. I haven’t had to think about it too often.
    * Times I have had to think about it were more so linked to “things” that just take a while to do, like reading a CSV. For those, beyond just making sure I wasn’t writing inefficient code, we just threw a hammer at it, like using Rust to process it, or trying to use IO streams instead of loading everything into memory. In a way, these streams where running in O(lg n) but that was an evaluation after the implementation, since I already knew the implementation was needed.
    * The only applicable aspect of this chapter is that I tend to double check things like nested loops to see if they are necessary, cause if they aren’t, it might be worth fixing. Or complicated recursive functions, yuck
* Gray
    * I remember learning this in college, but I’ve applied it very little
        * If I’m doing a classic sort, I’m using someone else’s algorithm and the speed is (usually) their problem
        * Haven’t had the need to write super performant code
    * Big O helps estimate how long things take but what can I actually do to make things faster? What’s the pragmatic thing to do when I’ve estimated how long a function takes to run?
        * Is it even worth it? Am I prematurely optimizing? I feel like estimating big O is easy, but telling if you’re prematurely optimizing is the more pragmatic thing to worry about
    * I feel like I don’t think about performance or speed as much as I do about events or the process constellations we have
* Lee


## Outcomes:


# Topic 40: Refactoring


## Examples:



* Jeff
    * They took the narrow definition of refactoring to only include situations w/ no change in behavior.
    * I think their garden metaphor is… ehhh… I haven’t entirely decided. At first I was very negative on it because it implies SO little planning. But I do like the idea that there are a lot of separate small processes coming together to serve an overarching purpose (feeding the family). But I still feel like it puts too little emphasis on the larger, overall architecture of things, that become harder to change if you get it wrong. That said, if they’re focusing on such a narrow band of work as: “make no behaviour changes”, then I suppose it holds up under that light.
        * Gardens can be thought of as farms, which need irrigation, crop rotation, large machinery for planting, harvesting, watering, fertilizing.
        * The selection of crops can be related.
    * I do like the idea that there’s no time like the present to fix something that could be better.
    * Their examples disagree w/ their def of refactoring though, in some cases. Specifically:
        * Outdate knowledge
        * Usage
    * I like the way they liken refactoring to pain management. Feels consistent w/ my notion of wanting to come to work every day. And the idea of poor code being a cancer that will metastisize if left unchecked is even better.
    * Shit, Martin Fowler apparently says not to add functionality at the same time… He knows his shit. But… I want to see the direct quote on this one. And that they got it from “UML Distilled” makes me doubt the veracity of the claim, and I can’t go look it up. I also don’t see how that fits the garden analogy, frankly
    *
* Kevin
    * “Don’t try to refactor and add functionality at the same time.”
    * Refactoring shouldn’t (usually) change the tests, they should still pass.
    * The concept of restructuring vs refactoring is new to me. I can see why there is a distinction.
    * “Take short, deliberate steps”, this usually manifests itself in each step gets it’s own commit, so it’s easy to revert if needed, I try to keep the tests passing between commits.
* Gray
    * “Don’t try to refactor and add functionality at the same time.”
        * I think this actually requires a bit of discipline to do properly, we’ve all seen something off in a PR and changed it alongside whatever feature we were putting in
    * “Take short, deliberate steps”
        * Very critical, don’t want to spend time doing a second refactor because all the tests are breaking and you have no idea how to track down what change you made that broke everything
    * It’s easiest to make the refactors now, even if they’re painful
        * Features steadily accrue over the course of a project’s life, which means that right now we have the least amount code that we need to deal with. If there’s an architectural issue it’s easier to address it before more features get added and further compound on any existing problems
* Lee
    * I like the gardening metaphor.  Small changes every day.
    * Distinction between Restructuring and Refactoring.
    * I’ve always liked the distinction between refactoring and adding behavior.  It makes both much easier to do, test, and review.
    * Super important: &lt;Step 3>


## Outcomes:


# Topic 41: Test to Code


## Examples:



* Jeff
    * If you write tests to design the software, and not to find bugs, why not chuck ‘em if you’re gonna refactor?
    * That demo example, where you pass the field in so you can test… the field, is a terrible violation of encapsulation.
        * “Thinking about writing a test for our method made us look at it from the outside, as if we were a client of the code, and not its author.”
        * False - It made you think about it as a test writer, not the client.
    * It goes on:
        * “So making your stuff testable also reduces its coupling.“
        * False - You’re just passing in and calling out what you’re coupling to. It’s injectable, but that doesn’t mean you’re less coupled. In their example, the exact amount of coupling remained, and in fact they made it worse by making the client inject the field name that should have been encapsulated by the test. So they ADDED coupling. Horrific.
    * And later about TDD…
        * “And that means you’ll always be thinking about your tests.”
        * Tests, according to them, are ways to write your code better. So… which is it? Are we thinking about the code, or the tests?
    * These analogies… I just don’t get - Describing Bottom up like:
        * The bottom-up folks build code like you’d build a house. They start at the bottom, producing a layer of code that gives them some abstractions that are closer to the problem they are trying to solve.
        * Bottom up folks I assume were starting at the simplest level and going up, like the later example of the TDD Sudoku guy. But here he makes it sound like he’s thinking foundationally, which I would have thought would have meant top-down. If not, what’s the difference between Top Down and Bottom Up in their mind?
    * I disagree that no one ever knows what the hell they’re doing at any time on any project for any scope. They state that like it’s just a true fact. I think there are shades of knowledge, and that not every task is like every other. Adding I18N was not as hard as adding rabbit, and the fog of war there was not the same size.
    * I don’t understand how their recommendation of go end to end is even related to “test to code”
        * “Tests can definitely help drive development. But, as with every drive, unless you have a destination in mind, you can end up going in circles.”
        * How is this compatible with the notion that you never know what you want
        * (That said, I do think Lee’s “thin slice, all the way through” advice has been solid, and we ignore that at our peril)
    * Norvig’s example of a sudoku solver, that they like, is a great example of NOT starting w/ tests. He just gets in and experiments w/ the problem, and plays w/ it til he gets it. So he figures out where he’s going, and then proceeds. They acknowledge that, while not recommending it
    * The DataFeed / LinearRegression example talks about testing the bottom components thoroughly so you will know that A is broken because of A. But we just talked about how bottom up development can result in creating software components that won’t actually solve the problem at hand, if you don’t know where you’re going first. Having those tests is valuable, but I do not think they’re being very clear in their guidance.
    * Lol, Confession time at the end.
        * a) I don’t write tests, because I don’t need to
        * b) I don’t write tests to design software, I write them to communicate with other developers
* Kevin
    * Yes
    * I agree with the sentiment of trying to write features in a TDD way, and resort to Test during if that proves difficult. It’s something I do, and usually produces well structured interfaces for the module created
    * The Top-down vs bottom-up approach seems to remind me of the plan down and code up approach I use that I mentioned earlier. Usually the planning top down can be jotted down in the form of test cases, that in itself is getting me to think of the big picture and start to “plan down”
    * Writing tests ensures the behavior consistently works, the nicest feeling is when you do a refactor or add a feature, and the tests still pass. Those tests and modules seem to be the best quality code (or it’s just a simple module)
* Gray
    * Testing will bring out the code smells if they aren’t clear already
        * Is your test aliasing too much stuff or stuff from across contexts? Might be opportunities for refactoring
    * “TDD: You Need to Know Where You’re Going” But do you?
        * I rarely have the full picture if I’m working on a big feature, but I should be able to figure out somewhere to get started. I know what functionality I want to have for a particular bit, no matter how big or small it is, I should be able to write the tests before I write the functionality at the very least
        * I find the way I work in reality is I’ll come up with a prototype or hack-y solution, refine it, test it, and iterate from there.
    * “As a contrast, Peter Norvig describes an alternative approach which feels very different in character: rather than being driven by tests, he starts with a basic understanding of how these kinds of problems are traditionally solved (using constraint propagation), and then focuses on refining his algorithm.”
        * Sounds hard but guy writes some really cool code so I believe this works for him, my brain is just not big enough (or experienced enough)
    * Testing Against Contract
        * This is why static typing is nice. Functions that have a type (or spec) have a contract implicitly, and the more refined your types are (structs) the better your contracts are. Functions that operate on higher-order types (structs) can offer more clarity than ones that operate on primitives
        * Might even be a way to auto-generate contract tests if function types/specs are known
* Lee
    * Testing from external behavior “what is it supposed to do” aspect, not “how is it doing it” allows easier refactoring. And up-front TDD style helps with this.
    * When doing TDD, you should be refactoring as you go.
    * I agree TDD shouldn’t become a religion.  Just like style guides.  They’re guidelines.
    * It seemed like they’re talking about two different things.  Architectural test/implementation (Top Down) and functional/tactical testing (at a function level).  Two different needs and approaches.  Doing a spike first to understand the overall picture makes things go faster
    * Spikes should focus on the highest risk
    * “... unless you have a destination in mind, you can end up going in circles.”
    * OTLP is a WAY better means of “testing in production” … we need to be aware of how to do this to exploit its capabilities
    * Truth: “Test During” is usually “Test After” becomes “Test only some” (but still better than “test none”)
    * “Test your software, or users will”.  And, to me, this is the scariest part of what we’re doing.  We’re going to make mistakes.  We’re going to encounter issues we never knew we had.  We “must” make our code tell us when we screwed up and how.  (Pattern-match, Process Exit, OTLP, Post condition checks).  If only I had a dollar for every “but that can’t happen”


## Outcomes:

	For some of our really important algorithms, I think we should consider adding post-condition assertions


# Topic 42: Property-Based Testing


## Examples:



* Jeff
    *
* Kevin
    * A property test on hedging seems like a great start to increasing our coverage and confidence with our hedging algorithm. Not sure if an elixir based property testing tool can be helpful here, just since I imagine the variants and invariants of hedging are complicated
    * Definitely a better chapter than the last 3
    * I only see value in this for relatively complicated pieces of code
* Gray
    * Property testing is testing against the contract
        * I don’t like how you litter your code with property tests in python, I think a better solution would be to use something like specs, then a macro could come in and create the property tests for you that could run as a part of your test suite
        * A macro would be nice because you could refactor functions to have different specs and never need to adjust property tests manually to reflect the behavior change (you still gotta fix those unit tests)
        * In elixir we get this at runtime with guards and pattern matching, but it would be nice to see where these violations occur before the code is ran.
        * Rust’s compiler does something similar to property checks but now you get them at compile time (Will tell you if a function’s return isn’t matching the function’s type for example)

    If something blows up in a property test, create a unit test for it

        * “Our suggestion is that when a property-based test fails, find out what parameters it was passing to the test function, and then use those values to create a separate, regular, unit test. That unit test does two things for you. First, it lets you focus in on the problem without all the additional calls being made into your code by the property-based testing framework. Second, that unit test acts as a regression test.”
* Lee
    * Yes please.  However, they’re time consuming to write and can interfere with refactors.  Should focus first on hardcore business algorithms like hedging where we can extract the code-under-test into its own module with a clear and stable interface.


## Outcomes:


# Topic 43: Stay Safe Out There


## Examples:



* Jeff
    *
* Kevin
    * Not sure I fully agree with the Password Antipatterns, some of them are definitely common practices.
    * Most of the information was pretty surface level, good for someone trying to get into understanding security. Definitely not enough to start going at it.
    * Didn’t know about the “$SAFE = 1” ability in bash. Don’t think it’s applicable to us
    * There’s an argument to be said we have willingly sacrificed a bit of security for easier partner configs by exposing things like their auth0 client id or the alpaca sweep and belay account ids
* Gray
    * “The first and most important rule when it comes to crypto is never do it yourself”
        * “the Pragmatic approach … let someone else worry about it and use a third-party authentication provider”
        * Security is hard, so it’s nice that the book acknowledges the best way to do something is to let an expert do it for you
    * Apply security patches quickly
        * I’ve got a bad habit of not applying updates
        * If you’re in a position where you are frequently applying patches, you should reconsider the options
        * Happens a lot with node JS projects via NPM. If you have a public node package sooner or later GitHub will let you know one of your dependency’s dependencies are compromised.
        * Good to really consider if a library is worth it because it’s an attack vector
    * The default settings should create the most secure environment
        * Good idea that I think goes unspoken a lot of the time
    * Belay security audit?
        * Best way to do something is let an expert handle it, so might be a good idea to have someone try to find weaknesses in our system. Curious as to how belay and the investor web (maybe even the proxies?) will hold up to scrutiny.
* Lee
    * Like many of the meatier topics of late, they’ve necessarily just touched the surface of a massive topic, worthy of its own book.
    *


## Outcomes:


# Topic 44: Naming Things


## Examples:



* Jeff
    * I’m generally very pleased with our naming practices. I feel like we take it seriously, put effort into it, and make changes when things aren’t going well.
    * Things we might consider changing:
        * Policies is the same concept in the Policies and Insurance context. Are we sure those contexts are truly distinct?
        * The change to Policies.Activated, where we’re just referring to the msg as Activated feels like a definite step backward in clarity to me.
* Kevin
    * We should have renamed Policies.Server -> Policies.ExpirationWatcher a while ago, that was one of those things we pushed back for a while
    * Renaming is a type of refactoring/restructuring that should be done on its own. Those are the easiest to review and get in quickly since the CI does most of the work here. Please no features next to renaming (unless it’s localized to a small area like a func, maybe module)
    * Glad we had those long discussions about changing all our instances of user to investor
* Gray
    * Naming things is hard
    * “There’s a well-established tradition that projects and project teams should have obscure, “clever” names”
        * PayPal names their projects after Harry Potter characters “Dumbledore is down again”
        * I don’t mind that we have very direct names for things
    * Consistency
        * I remember way back us discussing about naming things in your application after the business concept, that way when you have conversations about the business, they (should) translate 1:1 with what you’ve written in the code
    * Project glossary - think we should make one (at least before we onboard more developers)
        * Finance is confusing, insurance is confusing, a new product that exists at the intersection between the two is very confusing if you haven’t been immersed in it
        * Maybe we could get this with module docs? We have structs for business concepts, so giving them a doc and exporting that to html would make for a simple glossary
* Lee
    * That graphic reading-aloud thing was enlightening.
    * I REALLY don’t like the “Exception that proves the rule”.  Even w/i a company
    * Domain
    * Ideally, renames should be their own PR. Elixir is “typeless” and “simple” renames have a less-than-zero chance of causing subtle bugs.  Mixed in with bigger PR makes them really hard to find.  Ask me how I know.


## Outcomes:


# Topic 45: The Requirements Pit


## Examples:



* Jeff
* Kevin
    * I could spend an hour talking about the fact that clients never read requirement documentation. The TLDR is that Gray and I’s senior project was creating an application for a clothing company. We were forced by the CS department to use the waterfall model for planning (first mistake, I know). Our first 2 months was just creating the 50 page requirement document which our client approved and then we proceed to surprise the client with information unbeknownst to them due to them not reading the document. Our project turned into a shitshow.
    * The glossary got brought up again, I think it would definitely be useful to throw a ticket in there and add a glossary page maybe in slab (I know we haven’t used that thing in a while)
    * Requirement docs suck, no need. Probably better time spent trying to break down the requirements into user stories, coding can start usually immediately while the rest of the user stories come together.
* Gray
    * Requirements docs suck
        * Mostly a waste of time
        * You can never specify the best possible requirements because they will change over time, so either your doc is under specified and not giving a clear picture or you’ve over specified and those extra details are useless
        * At best it’s a legal document you use to cover your ass
            * An engineer should not be responsible for writing any legally binding document
            * Kevin and I had a senior project that never got finished but we walked away with our degrees because we had a req doc showing our client was trying to get us to do far more than we originally agreed
        * Devs don’t even use the damn thing! I’m just gonna go look at a card in click up to figure out what needs to be done
        * Makes more sense to just have an agile feedback loop. Be able to deliver updates to the client, have the client give you feedback in a timely manner, and be able to have the time to figure out the nuance of what the client really wants.
    * “Work with a User to Think Like a User”
        * I don’t know if spending a week to go work with your client is the best idea. If clients themselves don’t know the requirements, why would a week in their shoes give you any more clarity into the situation?
        * It just seems like having tight feedback loops between the devs and the client is more practical
        * Imagine explaining to your boss you blew off work for a week so you could go take calls at the help desk to really immerse yourself in the requirements
        * I’m not saying that there isn’t any benefit to first-hand experience, but you probably want to get that out of the way before any code starts getting written.
    * I like the note about expressing your business’s policies as configuration rather than as a behavior. It often gets told to you in a sort of “here’s how things are and they will never change” way but that’s just not true
* Lee
    * “Walk in your client’s shoes” is very effective.  And eye opening.  So Belay should give us all $100k to buy stock with.
    * I worked at Boeing after college.  Boy was that an eye opener.  STACKS of HUGE documents (that nobody read).  Useless waste of time and money.
    * VERY important: “Requirements are _need_”.  It really helps when creating tasks if we keep this in mind


## Outcomes:


# Topic 46: Solving Impossible Puzzles


## Examples:



* Jeff
    * I REALLY liked the notion of defining what the box truly is as opposed to thinking outside of the box. It just feels like it leads you down a more practical, prescriptive set of questions to ask.
    * Identifying the absolute constraints feels like another way of saying to go back to first principles when you hit a problem that’s REALLY hard.
    * One of the problems with letting bad code hang around, is that the constraint it imposes becomes the box that the next developer takes for granted.
        * Be open to taking a step higher, and reexamine the problem from there. A lot of times, problems that are intractable aren’t if you are willing to tear down a few more walls.
    * Lee yesterday reminded us to specify the goal of a ticket, not the task. Which i think is heavily related to this. In fact he challenged a how task just yesterday w/ the DLQ that helped us arrive at a solution requiring less code
    * Read the source. May the source be with you. Etc etc. A lot of times, reading the source helps me understand what the real constraints are, and how they can be bent
    * Thought it was interesting that they proposed trying to solve these problems by yourself. I think that a lot of the times we’ve been in conflict, it’s because we all came in with different ideas of how to solve a problem. But we were all in boxes that were too small in various ways. And by poking at our proposed solutions, reading the source, and brainstorming, the group eventually finds the real box.
    * I still think iterative development tends to look more like evolutionary development. Which encourages in-the-box thinking.  前に進むためには戻らなければならないこともある
        * “Sometimes, you have to go back to go forward.” - I just had google translate that to Japanese so it would seem more profound
* Kevin
    * Lots of repetition from Lizard Brain.
    * Taking a break is probably the best advice from this. This is getting harder to do with pair programming, so we should remember to push for it when needed.
* Gray
    * Your own preconceptions affect the way you write code and interpret constraints
        * I think pairing helps us get around this
    * How do I get better at figuring out the real constraints of a problem?
        * Brute force?
    * Eureka moments
        * Definitely real, solved many a bug while not at my computer
        * Probably not something you want to rely on, if you don’t have a path forward its best to make it known and switch to something else in the meantime
    * Gordian knot moments
        * Closest moment to this was way back when we were discussing the mechanics of owning and claiming a policy, what happens if a user has multiple policies on a stock? That was a big can of worms so we just said, you know what we’d rather not deal with that and set a new constraint that would allow us to have a simpler system
* Lee
    * This is largely a repeat of other chapters: Lizard Brain, Rubber Duck
    *


## Outcomes:


# Topic 47: Working Together


## Examples:



* Jeff
* Kevin
    * I don’t know how I feel about mob programming. At a certain point, I imagine there must be too many cooks in the kitchen. 4-5 seems like an upper limit for me, compared to them saying it’s a good starting point.
    * Collaborative programming is still one of those things I think works better for complex problems more than simple changes. When I’m going through and fixing tests, it’s easier for me to do that solo than with a pair.
    * Focus is something that gets burned through much faster when pair programming, we should manage it well if we want to make this a continued practice. Breaks or splitting up on easy work is something I want to do more of to maintain focus and pace.
    * Automation is another one I do for sure love
    *
* Gray
    * “The inherent peer-pressure of a second person helps against moments of weakness and bad habits”
        * I definitely feel less inclined to prototype or jump right into the code to figure out how things work but I feel like that’s helping me make more deliberate choices that we can have a conversation about
    * “The developer acting as typist must focus on the low-level details of syntax and coding style, while the other developer is free to consider higher-level issues and scope."
        * If I’m not typing I find that I’m also looking at the syntax, picking out typos and whatnot but I’m wondering if I should be focusing more on the bigger picture in these moments.
    * Mob Programming
        * I feel like a mob loses effectiveness as it grows. If the goal is to have tight collaboration with live coding, more people doesn’t make the loop tighter. The more people on a problem, the less likely any of them are to be fully invested in the problem and I feel like you’d get better results with a smaller mob.
        * If there’s a hard problem, mob to figure out a direction, then pair on the solution
* Lee
    * LOVE the opening quote.
    * “... geographically distributed teams are shown to tend toward more modular, distributed software.”  Interesting
    * When pairing, I’ve a bad habit of interrupting. Please call me on it.


## Outcomes:


# Topic 48: The Essence of Agility


## Examples:



* Jeff
    * I’m curious on this. I am terrible (or great depending on your perspective. I personally think less is more, but maybe not as little as I have us do) about process, and so we fall off on sprint planning a lot and on sprint reviews. Haven’t done one in awhile. If this discussion runs short, I’d be curious to just transition into something of a retrospective on our last few months
    *
* Kevin
    * I think we’ve always hit this one on the nail.
    * Due to our small team’s nature, it’s “easier” to follow the values from the agile manifesto, the bigger we get, the more of a problem this becomes. In bigger companies, this is where I think those agile focused team member positions start to pop up.
* Gray
    * The feedback loop is what makes agile work
        * Goal of the feedback loop is to take what you have, improve it such that it can better meet it’s goals, and still have it all work at the end of the loop
        * You have a feedback loop with yourself when you code (Tracer -> Prototype -> Feature)
        * You have a feedback loop with other devs in PR reviews
        * You have a feedback loop when you deploy to users (Direct feedback, analytics)
    * I think in general we have pretty tight loops, but stability is something that I worry about
    * Agile-in-a-box
        * I worked for PMI, the project management institute, and they used an agile process only in name. What they really wanted was a way to quantify how much work you do as a dev with sprint points and velocity, and then use that as a metric to say if something is going well or not, and they hammered agile until it fit their expectations.
        * No one even really got to work in an agile environment anyway because no one did the work assigned to them, everyone wanted to work on “hot-bricks” which are priority tasks that should have been done yesterday.
* Lee
    * Work out where you are
    * Make the smallest meaningful step toward where you want to be
    * Evaluate and fix anything you broke


## Outcomes:


# Topic 49: Pragmatic Teams


## Examples:



* Jeff
    * Definitely, 100% agree w/ the pieces here about no broken windows, no boiled frogs, and owning your own employability
    * My hope is that our project never _FEELS_ like old systems maintenance.
    * I think we can do better w/ rotating pairs and making sure we’re all well-rounded in all the things as much as possible
    *
* Kevin
    * We fell out of the habit of scheduling our sprint tasks. Things were moving faster and changing even faster, so we dropped that. We should bring that. I usually liked the idea of particular tasks we want to work on in a sprint just get moved into the current sprint.
    * Automation is huge for us, we are a small team, we need to automate as much as possible to be as efficient as possible. Without automation, we would just be doing constant maintenance. We nail that one
    * We are also definitely fully functional (for now), just due to the fact that we don’t have a QA, frontend, or backend developers. We are all somewhat versed in most aspects of the app(s)
    * I’m not sure how I feel about the “Schedule your Knowledge Portfolio” section. On one hand I agree that not scheduling in maintenance or skill improvement/tech experiments will lead to them never happening. But the idea of getting back to something when you are free is still a necessary choice when we have more pressing stuff to do. We are a small team, so tasks are less likely to be lost to our backlog, but it’s still happening. Having the CTO on our team also helps make sure the planning is taking into account these non-feature tasks.
* Gray
    * Thought it was interesting we got a chapter about managing a team. Nice change of pace
    * I feel like a lot of these are things we already do or are problems for bigger orgs
    * Broken windows - Generally I feel like we’re good about this
        * Sometimes we go so far as to fix broken windows in other people’s code (fixing Otel decorator tests)
        * But sometimes we seem to ignore things in our own backyard like failing smoke tests
    * Boiled frogs - Jeff is good about making requirements known, if something happens in a board meeting, he tells us and we can respond accordingly.
        * BigOrg problem we thankfully don’t have
        * I think the real thing that would boil us is complacency either with broken windows, less than stellar test coverage, or stability
    * Schedule your knowledge portfolio
        * I feel like when topics like these come up we always talk about how lunch and learns would be nice but then we don’t do them
        * But honestly I think we actually do allocate a lot of time for this knowledge, we just do it after standup instead of after lunch
* Lee
    * “... merely strangers temporarily sharing a bus stop in the rain.”  … Somebody went to the Jeff Deville School of Analogies
    * Their comments about teams really hit home.  Like any other relationship, if things are bad and they’re not willing to change … bounce.  It’s SOOO frustrating being on a team that doesn’t value quality and you can’t fix it.  You’ll just get loaded with more shit.
    * “Scheduling Your Knowledge Portfolio” is HUGE.  So many ways to do this.  Important for teams, the code, and the individual.  Take time to learn.  Extend your IDE, read blogs, read core team code.  Dive into Sentry reports.
    * Don’t know what dope they’re smoking with “... People look forward to meetings with them …”  Who the heck looks forward to meetings?
    * We can do better with pair rotation.  There’s a balance we need to acknowledge where everyone is a generalist and nobody has depth.  Can have a huge and costly impact on the codebase


## Outcomes:


# Topic 50: Coconuts Don’t Cut It


## Examples:



* Jeff
    * “Why” not “what”. This is why I don’t put much stock into what other folks say unless they can either a) explain it to me, or b) they’ve banked so much credibility w/ them that I can assume that their rationale would match or exceed my own.
    * This is a thing that I feel we’re doing well with. Balancing avoiding NIH, and blindly reusing a tool I don’t understand.
* Kevin
    * Taking a look at their delivery time vs iteration period, we are doing pretty damn well. Maybe just short of fully CD.
    * It seems Clickup seemed to slow us down as the number of tasks shrank due to us reaching/approaching our MVP
    * Curious to learn what “Kanban techniques” are, I thought that was just the name for that style of board
* Gray
    * There’s definitely a mentality of “big company did it so we also need to”
        * PMI was like this, very focused on delivering things that are fashionable
        * I was told to develop a project with AI by myself with no resources and present it in a month at their company faire, ended up just putting an iframe in electron that pointed to a Microsoft sentiment analysis demo. No one knew what they were looking at but since it was AI everyone was impressed.
    * I’ve seen the opposite though where it becomes “we’re big company so we can do whatever”
        * SAP has their own in house JS UI framework called SAP UI5, and you haven’t heard of it because it’s a massive piece of shit
        * Whereas just using something like more common like angular would have made everyone far more productive. Anything where you don’t need to maintain and document your own library would have been an improvement.
* Lee
    * Like everything we do, it’s all about having tools in our toolbox and knowing how and when to use them.  Languages, algorithms, architecture, testing, processes.  Understanding the tradeoffs and what’s available


## Outcomes:


# Topic 51: Pragmatic Starter Kit


## Examples:



* Jeff
    * Felt redundant to other chapters to me
    * I do like the idea of them weighing in on the value of automation over and over though
* Kevin
    * Property based testing seems to be a good investment, along with the needed e2e tests for trading.
    * Good to know we are using a pragmatic starter kit
    * The idea of using tags to kick off a deploy to sandbox and live seems like a cool idea, we aren’t versioning our api currently, but eventually I imagine we will want to mark it as 1.0.
    * “And once you introduce manual steps (just for this one part…) you’ve broken a very large window”. Good thing to keep in mind, I’ve definitely made those types of arguments in devops land, I just did this morning regarding CloudAMQP and terraform.
* Gray
    * No need to tell us twice, we’re already doing this
    * There’s no pragmatic way to work with other people (or even with yourself honestly) without some form of version control
        * But I actually don’t think it ends at just using it, you need to use it properly
        * Worked at a place where they used subversion instead of git. Problem wasn’t subversion, it was the fact everyone didn’t use PRs and just merged shit in whenever. Most of your day working there is just resolving conflicts that shouldn’t really be happening.
    * I like their note about how testing state is more important than testing coverage
    * I think chaos monkey is pretty interesting, and you could probably make your own mini version of it in elixir pretty easily. Not sure if it really gives us value but would be cool to actually have a way to test resilience of the system.
* Lee
    * Chaos Monkey would be a very valuable thing to introduce
    * Reiterate the need for regression testing


## Outcomes:


# Topic 52: Delight Your Users


## Examples:



* Jeff
    * This has always felt like has been the thing that sets me apart from really good, fast developers.
* Kevin
    * Another thing that we get cheaply for being a small startup. We can’t avoid understanding the business goal due to it being so in our face, and not knowing it could cost us our job. I imagine it will become harder and harder to enforce as we grow
    * Eventually our customer will be different, and we will need to serve them, aka the partners not the partner’s investors
* Gray
    * At the bigger orgs I’ve worked at it’s common to be completely disconnected from the expectations of the users
        * If the software is a means to an end how can we deliver anything of value without understanding what that end is?
    * But what do we do when there’s no way for a user to have an expectation?
        * We did a lot of analysis of the our original user experience (Reggie) now it’ll be interesting to see if the investor web is meeting it’s expectations
        * But what are those expectations? How can we meet the expectations of users and investors for a product they’ve never interacted with before? When it was just Reggie it was pretty easy because Reggie is a dev consuming an API, but now we have users purchasing policies. What do they expect? What do they even think of the product?
    * A part of why I like frontend work is because you can almost intuit your expectations: “If I were a user, what would I want to do with this, and how should it get done?”
        * Emphasis on “almost”
* Lee
    * “How will you know that we’ve been successful …”  Just like an `asset` in a test case
    * Providing the solution they “need”, (usually) not what they want (or asked for)


## Outcomes:


# Topic 53: Pride and Prejudice


## Examples:



* Jeff
    * I’ve never thought about this in QUITE this form, but the idea of thinking of ourselves as craftsman is an interesting one. That said, all things are relative. It makes sense for belay-api to be built by craftsmen. But for a 1-off task for some reporting, it would be better to just get it done.
* Kevin
    * Whenever we talk about any type of piece of code, we take communal ownership over it, we always say “we decided this” or “we made this decision”. I think that helps us both feel good about the right decisions, and not as badly for the wrong ones since they aren’t pinned on one person.
    * Pride of ownership of code I think can be a motivating factor for me, it’s nice to see a feature that I created along with other team members and say “I did that”
* Gray
    * Communal ownership is a good model
        * We’re all responsible for the quality of the code our team produces, on some level we all need to take ownership of all of the code
        * We all own the code but individuals own the expertise — when we put our heads together to solve problems we’re not bringing bits of code that we own into the conversation, we’re bringing the knowledge writing that code facilitated
    * Pride of ownership
        * I don’t think pride is pragmatic
            * If I’m proud of the code going into PR I’m gonna feel hesitant to implement code review changes
            * If I’m proud of code that never ends up never getting deployed, I’m gonna feel really bad that it never saw the light of day
        * I talk a lot about balancing emotional investment with my other friends who are professional programmers. The consensus is that we feel as if we do better work when we are emotionally invested, that is to say, have pride in the work we’re doing
            * But it feels like shit when you don’t deploy or when things go south. The pain is proportional to the investment. What’s the line to draw?
        * Maybe I’m conflating pride with ego or emotional investment with the code we write. Don’t get me wrong, we’ve built the ‘impossible product’ at belay, that’s worth taking pride in. But taking pride in all of the little steps along the way, even the ones that turned out weren’t taking us in the right direction, feels like it would set me up for disappointment.
            * Investor web is an example of this. I’m happy with what I’ve made but I’m more proud of the ‘enabling’ role it’s had at belay. At the end of the day, the investor web is temporary. And I think that’s how you balance it.
* Lee
    * Have always felt that the term “craftsman” is the best description of our job … at least for now.  Who knows what AI and the future brings?
    * Pairing can overcome the reluctance to change code you’re “proud of” (aka, made too much investment in before getting team feedback)


## Outcomes:


# Interesting Questions:


## Orthogonality


### Ex 1


```
    "You're asked to read a file a line at a time. For each line, you have to split it into fields. Which of the following sets of pseudo class definitions is likely to be more orthogonal?"
    class Split1 {​
        constructor(fileName)    # opens the file for reading​   def readNextLine()       # moves to the next line​
        def getField(n)          # returns nth field in current line​
    }
    or​
    class Split2 {​
        constructor(line)        # splits a line​
        def getField(n)          # returns nth field in current line​
    }"
```



## Ex 2


```
    "What are the differences in orthogonality between object-oriented and functional languages? Are these differences inherent in the languages themselves, or just in the way people use them?"
```



## Reversibility:


```
    "Time for a little quantum mechanics with Schrödinger's cat.  Suppose you have a cat in a closed box, along with a radioactive
    particle. The particle has exactly a 50% chance of fissioning into
    two. If it does, the cat will be killed. If it doesn't, the cat will
    be okay. So, is the cat dead or alive?  According to Schrödinger, the
    correct answer is both (at least while the box remains closed).
    Every time a subnuclear reaction takes place that has two possible
    outcomes, the universe is cloned. In one, the event occurred, in the
    other it didn't. The cat's alive in one universe, dead in another.
    Only when you open the box do you know which universe you are in.
    No wonder coding for the future is difficult.
    But think of code evolution along the same lines as a box full of
    Schrödinger's cats: every decision results in a different version of
    the future. How many possible futures can your code support?  Which
    ones are more likely? How hard will it be to support them when the
    time comes?
    Dare you open the box?"
