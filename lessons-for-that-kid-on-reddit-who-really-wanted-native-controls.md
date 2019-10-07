Title: A brief note to young players
Blurb: A followup to a note I left to someone new to this game.
Tags: Philosophy,Programming

A while ago I left this note to someone new to the world of development who was frustrated with the speed at which new features are added to web browsers.

<blockquote cite="https://www.reddit.com/r/programming/comments/93502u/blink_intent_to_deprecate_and_remove_shadow_dom/e3b4g1v/">
<p>Yeah this part is really starting to piss me off to be honest, Firefox STILL doesn't support components, Firefox STILL doesn't support HTML 5.1 dialogs (native modal support so that we don't have to keep doing modals in JS), I've literally been waiting for years for other browsers to catch up with Chrome so we can finally start using these technologies. I really do like Firefox but lately it seems supporting the latest standards no longer seems their priority and that makes me sad.
</p>
</blockquote>
<cite>– robvl</cite>

And my response:

<blockquote cite="https://www.reddit.com/r/programming/comments/93502u/blink_intent_to_deprecate_and_remove_shadow_dom/e3boab6/">
<p>"seems their priority"

You talk as if Firefox devs are these nebulous far off being that are somehow separate from us. I get that you like firefox and want it to be great and are coming from a place of frustration, but please (for everyone reading this) remember that Mozilla it made up of real people with real budgets and timelines and issues and competing attentions and priorities. It is hard to make this whole thing work. There are only so many hours in the day.

Since you really like ff, could you spare 10 minutes to report a bug or write some tests to help out? Review the spec or comment on the current implementation. There must be something that you can do that will help get everyone closer to these features.

Again, I know it is borne out of frustration but there is so much you can do to help everyone get past it.
</p>
</blockquote>

<cite>– me</cite>

While my original point still stands (be patient, dev is hard and expensive, help if you can) I feel there is a point not said.

To which: the perceived issues you are dealing with are Sysiphusian in nature. Having the solution in place for this issue wont magically solve all your problems it only kicks the can down the road. You will just run into the next problem and the next one after that and the one after that one too until the day you stop working.

I would argue that it is a sign of a maturing dev when they can accept this reality and work with it to produce software and solve problems rather than bemoan the situation and not move forward.

My personal bug bear was inline/first-class citizenship of SVG that came with Firefox 4.0. What a glorious day that was when it shipped! We could have SVG everywhere and in everything, birds would sing and babies would be born. So I went about and put SVGs directly in to pages.

Nothing changed.

Previous to that I needed to either put them in via <code><object></code> tags or equivalent to have them show up. It was an annoying extra layer and hoop to jump though that took extra effort and tooling to solve. But solve it you do and move on to the next problem. And you do move on. That's the secret here, you deal with what you have in-font of you, make compromises, push back where you can and ultimately accept your fate. Getting angry and frustrated, while cathartic, doesn't help with the situation.

And it is Sysiphusian task as because as soon as you overcome one challenge the next rears it's ugly head. If you can manage to push long enough and hard enough, you can ship some pretty OK software maybe.

Back to the original poster: what would having the native controls then lead to? One less dependency in the page? While that is a nice ideal to have, the problem is solved enough that it is time to move on to the next issue.

Which leads back to my next issue. Having <code>Wallclock</code> implemented for SVG would make a project I am working on so much easier and cleaner to implement but alas we are not getting it in 1.2 but maybe we are in 2 (maybe I should do it if I were to listen to my own advice).

So back to the mountain...



