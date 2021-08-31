# What I learnt from working in science

The idea of Science is wonderful. It removes the ambiguity from a situation and allows us to move forward as individuals and as a society as a whole. Working as a scientist changes the filters in which you see the world. These are some filters I have noticed change in myself, especially in the way I work in IT.

## Measure. Measure, measure, measure...
It is impossible to make informed judgements, and the best decision about your situation if you don't measure. There are a few different ways this applies to development work and project management.

After a while in IT work you gain an intuition as to what might be the cause of a slowdown or error or perhaps which design might have the best efficacy for users needs. These are guesses however, and prone to all sorts or human biases and prejudices. There needs to be systems and processes in place that are able to overcome or bypass these failings of human nature.

You might have monitoring of services in place you might let you see that a particular process has silently failed which is blocking the resources of another visible process. It could be all to easy to spend far too much time looking at the second process when the whole situation could be short avoided by proper measurement.

By testing with representative users you can gauge actual problems they need solved rather than the assumptions that the original stakeholder imagined (they might be the same but unless you test, you do not know.)

Having adequate test coverage allows you to pinpoint where errors have been introduced. By removing the ambiguity in a system you allow said project to make informed decisions which mean you can complete the project/task faster with more confidence in the work.

### Understand what you are measuring
The flip side of measuing everything is that your signal-to-noise ratio can go to hell if you are not absolutely judicious about how you use the data. It is no use measuring without understanding what it is that measurement gets you and what it means in a larger context. To say another way, there is a risk of miscomprehending the results of a test you might have run or inferring too deeply about what the data is telling you which can lead you down an incorrect path.

### There is a cost
Obviously there is overhead to do this: time and resources. How much effect does the monitoring have on the performance of the server? Is it impacting the system significantly?
Is there time factored in to the delivery schedule to iterate with representative users (one would hope so)? Are they available?
If you are using tests in your code, the data shows that (provided you are doing sane things) you are probably more efficient [citation needed] when using tests…

## Remove your ego
Get over the idea that you are a good and worthy person because of the work you have done. The work itself should be able to stand on its own legs and tell that story for you. By removing yourself from the equation you do two import things: open yourself up for improvement and allow yourself to fail, both of which will allow you to grow in constructive ways. You can be proud of excellent work, of course, but connecting your self-esteem to your work output is destructive. You are not your code.

By separating yourself from the work (the new super fancy algorithm, the amazing design, your research results etc) you open yourself up to inputs and perspectives form others that might improve or enhance said work.

This is not saying that you should blindly incorporate all of their feedback or ideas, far from it; it is saying that you need to have the work tell its own story and be able to grow by itself, capable of standing up to and rejecting the bad ideas and being able to assimilate the improvements without your own ego rejecting them because they did not come from you.

Your role is more of a steward at the helm, steering the ship through the treacherous waters of project delivery rather than dictator trying to hang on to power at all costs.

This can be hard and even (physically [citation needed]) painful but is necessary to do, especially in light of the second idea: what happens when a project fails. If a project succeeds because it is part of you, part of your very worth, how much of a blow must it be to your very self if said project fails? Is your worth now tied to a failed project? Are you now worth less? Separating yourself form the work and allowing it to stand on its own also allows it to fail on its own.


## Nothing gets done alone
Unless you are [a 19th century aristocrat][ada_lovelace] then you will be working with others to get things done. You will need to know how to work in a team. While this is obviously not unique to Science, visibility of how much work is produced by teams is certainly more opaque in technology circles.

Outside the immediate line of sight of coworkers and teammates, for which you will have to have adequate communication skills and take a shower every day, you will need to function in the wider tech world. [Learn how to ask good questions][zellwk_question], be appreciative of the work others do that lets you do yours [citation needed (open source contributions)] and generally just do not be a cock. It gets lonely out there by your self and take a long time to move a mountain.

## Scientists vs Science fans
Recite π to as many decimal places as you can. I'll wait.

Now go ask a scientist the same question and compare the results.

It is likely that a scientist's response will be much sorter than a science fan's. By a long way. This highlights the difference in approaches and desired outcomes.
Science fans are satisfied knowing 'stuff about things', which is absolutely awesome. I am by no means attempting to disparage these people. Scientists on the other hand have a different desired outcome: a deeper understanding with larger… structures used to construct and model with.

The parallel also exists within technology also: people who spend a week on one framework, espouse it as the greatest thing ever then move on to the next thing. The other people are looking through this noise and working with the data structures and algorithm.

It is of course much more nuanced and stratified than this, but it pays to be aware of the distinction and recognise which camp you are in and which you want to be in.

## Final thoughts
Be connected of the world you live in, you need to live in it so do not make it any harder than it needs to be.

[zellwk_question]: https://zellwk.com/blog/asking-questions/

[ada_lovelace]: https://en.wikipedia.org/wiki/Ada_Lovelace "Not much you cant be when filthy rich"