---
title: Taste as Architecture for Components
excerpt: A reflection on taste, creativity, and developer empathy when building front-end components that stay open, fast, and useful.
date: 2026-02-06
---

<section>
  <div class="row">
    <div class="col-4 col-md-8 col-lg-12">
      <article class="brixs-article">
        <header>
          <h1>Taste as Architecture for Components</h1>
          <p>2026-02-06 • Creativity, taste, and APIs • 18 min read</p>
        </header>

        <section>
          <h2>Introduction</h2>
          <p>
            I work in the design system of the largest bank in South America. It has been four years
            of building components for people I may never meet, but who I think about every single
            time I name a prop, design a slot, or decide how an event should bubble. Every decision
            is a conversation with a stranger. I do not know their project, their deadline, their
            frustration level. But I imagine them. I imagine them opening a doc page at 11pm trying
            to ship something before morning. That person deserves an API that does not waste their
            time.
          </p>
          <p>
            The first question I always ask myself is simple: if I am the developer consuming this
            design system, how should this thing be used? And if the answer is not immediately
            obvious from the markup, something is wrong. Not with the developer. With me.
          </p>
          <p>
            That question is about taste. Not taste as in visual preference or trend. Taste as in
            the invisible architecture behind a component. It is not the public API itself, but
            all the decisions that make that API feel obvious, honest, and durable. I think of it
            like a building. The component is the building, and the developer is the person who
            lives inside it. They should not have to study the blueprints to find the door.
          </p>
        </section>

        <section>
          <h2>Taste is a constraint, not a style</h2>
          <p>
            Most people hear "taste" and think of decoration. Something visual. Something you add
            on top. But in practice, taste is the opposite: it is a constraint. It is a refusal to
            overfit. A refusal to hide native behaviors when they are already good enough. A refusal
            to make everything a special case because someone filed a ticket.
          </p>
          <p>
            I genuinely do not like complicated things. I have no patience for them as a user and
            no interest in building them as an engineer. What I want is something highly optimized,
            open, composable, and customizable. Something where the developer can look at the markup
            and understand what is happening without reading a novel of documentation first. That is
            taste in engineering terms: the discipline to subtract.
          </p>

          <blockquote brixsBlockquote="info">
            <span brixsTag>Rule:</span> If a component can be understood by reading its HTML, the API is
            probably honest.
          </blockquote>
        </section>

        <section>
          <h2>Developer empathy as the first spec</h2>
          <p>
            I always design from the outside in. Before I write a single line of implementation, I
            ask: how should this be used? If I am the person consuming this system, what would I
            expect? The answer should be readable right there in the markup. If the usage is not
            obvious, the component is not done. I am not interested in clever. I am interested in
            clear. That is why I will always prefer simple, composable pieces over magic wrappers
            that do too much behind the curtain.
          </p>
          <p>
            In practice, that means obsessing over default behaviors, escape hatches, and learning
            cost. It means using native events whenever possible, only overriding when there is a
            real, demonstrable need. It means naming props in a way that a developer who has never
            seen the component before can guess what they do. It means being boring on purpose,
            because boring is fast to learn and hard to break.
          </p>
        </section>

        <section>
          <h2>Is there a W3C reference for how components should be built?</h2>
          <p>
            People sometimes ask me if there is a spec that tells you how to build components the
            right way. The honest answer is: not really. The W3C gives us the Web Components specs,
            Custom Elements and Shadow DOM. They define the mechanics. They tell you what the
            browser can do. They do not tell you what you <em>should</em> do. The spec does not
            tell you if a component should be open or closed, light DOM or shadow DOM. It does not
            tell you when to expose an event or when to wrap one. That gap between mechanics and
            meaning is exactly where design systems live. That is our job.
          </p>
          <p>
            For reference, see the
            <a class="brixs-link" href="https://www.w3.org/TR/custom-elements/" target="_blank" rel="noopener noreferrer">
              W3C Custom Elements
            </a>
            and
            <a class="brixs-link" href="https://www.w3.org/TR/shadow-dom/" target="_blank" rel="noopener noreferrer">
              Shadow DOM
            </a>
            specifications. They are tools, not answers. The answers come from taste.
          </p>
        </section>

        <section>
          <h2>Light DOM or Shadow DOM?</h2>
          <p>
            My default is open and predictable. Always. If the component is highly interactive, if
            it has rich APIs and the user needs to wire things together, I want it to be open to
            native events and native behaviors. That often means an open shadow root, or even light
            DOM when the styling model allows it. I am not precious about encapsulation for its
            own sake. Encapsulation is a tool, not a religion.
          </p>
          <p>
            When the component is deeply customized and needs to guarantee strict internal
            structure, I may choose shadow DOM and carefully surface the events that matter. But
            that is the exception, not the rule. And even then, I hate hiding things from the
            developer. The key is to avoid false isolation. If the user needs to integrate with
            forms, focus management, or accessibility tooling, do not hide the native elements
            from them. If you take something away, you owe them a replacement that works at
            least as well as what you took.
          </p>
          <blockquote brixsBlockquote="success">
            <span brixsTag>Guideline:</span> If you override a native behavior, re-expose it in a way
            that feels native to the platform.
          </blockquote>

          <p>
            This is not an aesthetic choice. It is an operational one. The DOM mode you pick
            changes how assistive technology discovers labels, how form association works, and
            how focus escapes nested controls. A visible, predictable tree is a feature. If a
            design demands isolation, you have to replace every behavior you hid: expose the
            same roles, the same states, the same events, and make absolutely sure that keyboard
            and screen reader flows stay intact. If you do not, you have broken the web for
            someone, and that is never acceptable.
          </p>

          <pre brixsPre><code class="language-typescript">// Example: wrap a prototype getter/setter (indeterminate)

const input = document.querySelector("input[type=checkbox]") as HTMLInputElement;
const proto = Object.getPrototypeOf(input);
const descriptor = Object.getOwnPropertyDescriptor(proto, "indeterminate");

if (descriptor?.get &amp;&amp; descriptor?.set) {
  Object.defineProperty(proto, "indeterminate", {
    configurable: true,
    enumerable: descriptor.enumerable,
    get: () =&gt; descriptor.get!.call(input),
    set: (value: boolean) =&gt; {
      descriptor.set!.call(input, value);
      if (value) {
        input.checked = false;
      }
    },
  });
}
</code></pre>

</section>

        <section>
          <h2>Accessibility as architecture</h2>
          <p>
            I need to be very clear about this: accessibility is not a compliance layer. It is not
            the thing you add at the end to pass an audit. It is the core load-bearing structure of
            everything I build. If accessibility breaks, the component breaks. Full stop.
          </p>
          <p>
            WCAG is a set of outcomes, not a design aesthetic. It pushes me to ask real questions:
            can someone use this with a keyboard only? With a screen reader? With high contrast
            settings? Without losing any information? If the answer to any of those is no, the
            component is not done. I do not care how beautiful it looks or how clever the API is.
            It is not done.
          </p>
          <p>
            Here is where it gets interesting, though. When a design request conflicts with
            accessibility, I do not treat it as a wall. I treat it as a design constraint. And
            constraints are where the best work happens. I look for the smallest change that
            preserves the designer's intent: adjust spacing instead of shrinking hit targets,
            add a visible focus ring that matches the brand identity, use motion that can be
            reduced via <code>prefers-reduced-motion</code> without losing meaning. The goal
            is always to keep the vision without making it exclusive. Design that excludes
            people is not good design. Period.
          </p>
          <p>
            I keep a practical checklist tied to the platform: WCAG 2.2 success criteria, the
            WAI-ARIA Authoring Practices, and the native HTML specs. Those references define the
            non-negotiables: focus order, name/role/value, input modality parity, sufficient
            contrast, target sizes. They are not constraints on creativity. They are the conditions
            that let design survive contact with real people in real situations. People who deserve
            the same experience as everyone else.
          </p>
          <p>
            The best components carry their accessibility in the markup itself. Proper labels,
            helpful descriptions, respectful defaults. When you build that way, the system
            becomes safer, faster to adopt, and harder to break. That is taste operating at its
            deepest level: not just how something looks, but how it behaves under pressure, for
            everyone.
          </p>
          <p>
            References:
            <a class="brixs-link" href="https://www.w3.org/TR/WCAG22/" target="_blank" rel="noopener noreferrer">
              WCAG 2.2
            </a>,
            <a class="brixs-link" href="https://www.w3.org/WAI/ARIA/apg/" target="_blank" rel="noopener noreferrer">
              WAI-ARIA Authoring Practices
            </a>,
            <a class="brixs-link" href="https://html.spec.whatwg.org/" target="_blank" rel="noopener noreferrer">
              HTML Living Standard
            </a>.
          </p>
        </section>

        <section>
          <h2>Influence is a system</h2>
          <p>
            I do not separate my engineering taste from the things that shaped it. They are the
            same thing. The way I think about a component API comes from the same place as the
            way I listen to music or look at a room. Let me explain.
          </p>
          <p>
            James Turrell builds rooms where the only material is light. There are no objects to
            look at. The room itself is the experience. You walk in and your perception shifts.
            The walls dissolve. The color feels physical. He taught me that light is not decoration;
            it is structure. And that changed how I think about UI state. A focus ring is not an
            ornament. A loading indicator is not flair. They are the structural light of the
            interface: remove the noise, let the state be readable, make the focus feel inevitable,
            and let layout guide the eye without instruction. Turrell does not tell you where to
            look. He makes it impossible to look anywhere else. That is what a well-built component
            should do.
          </p>
          <p>
            Miles Davis recorded <em>Kind of Blue</em> and changed music forever, not by adding
            more notes, but by leaving space. The silence between the notes is not absence; it
            is tension, it is breath, it is trust that the listener will stay. His restraint
            haunts me every time I design an API. The methods I choose not to add matter as much
            as the ones I do. The props I refuse to invent, the defaults that let the user play
            without needing a manual first. Taste is the space you leave so others can move. If
            every beat is filled, there is no room for the musician. If every prop is prescribed,
            there is no room for the developer.
          </p>
          <p>
            Alexander McQueen made clothes that looked like they were about to come alive. His
            shows were theatrical, raw, sometimes uncomfortable. But the structure underneath was
            immaculate. The seams were visible because the craft was the story. He showed me that
            structure can be dramatic without being chaotic, that power does not need to be hidden.
            In a design system, that translates directly: expose the seams. Let slots, events, and
            native elements be honest about how the component works internally. Do not hide power
            behind abstraction. Stage it. Let the developer see the architecture and trust it.
          </p>
          <p>
            Chrome Hearts is heavy, tactile, and completely unapologetic. Every piece feels like
            it was made to survive something. They value material and finish above trend. That
            gives me a standard for durability in software: components should feel weighty in a
            good way. Stable under edge cases. Resistant to collapse when they meet real data,
            real users, real network conditions. I want someone to use a component and feel that
            it was built to last, that it was not assembled from shortcuts. Beauty in Chrome Hearts
            is always a consequence of strength. Same with components. If the foundation is solid,
            the surface takes care of itself.
          </p>
          <p>
            Issey Miyake spent his life studying the space between fabric and body. His Pleats
            Please line is one piece of fabric that becomes many shapes without losing its identity.
            You fold it, you stretch it, you pack it flat, and it comes back the same. That idea
            lives in my component architecture: a core that adapts to multiple contexts without
            fracturing into variants. It is modularity with a soul. The component does not need a
            new version for every context. It needs a structure flexible enough to fold without
            breaking.
          </p>
          <p>
            When all of these influences meet engineering, a design system stops being a catalog
            and becomes a language. A real language, with grammar and rhythm and silence. The
            principles stay consistent: show the structure, honor the user, design for use and
            not for screenshots. Accessibility is part of that language from the first word, not
            a translation stapled on at the end. If it does not work for everyone, it is not
            finished. It is not even started.
          </p>
        </section>

        <section>
          <h2>Virgil Abloh and the 3% change</h2>
          <p>
            Virgil Abloh said something that I carry with me every day: creativity is a 3% change.
            Not a total reinvention. Not burning everything down and starting over. Just a precise,
            meaningful adjustment that changes how we see the original. Three percent. That is it.
          </p>
          <p>
            In component design, that 3% is everything. It is the difference between a raw
            <code>&lt;input&gt;</code> and a high-quality text field. The default spacing that
            feels right without anyone noticing. The keyboard flow that just works. The error
            message that appears exactly when you need it and not a millisecond before. The
            transition that respects <code>prefers-reduced-motion</code>. None of these are
            radical inventions. They are edits. Careful, disciplined edits on top of what the
            platform already provides.
          </p>
          <p>
            Virgil understood something that most engineers miss: the original is already good.
            HTML is already good. The browser is already good. My job is not to replace them.
            My job is to make a 3% change that elevates the experience without distorting it.
            That small change is taste. It is also restraint. It is an edit, not an explosion.
            And it takes more courage to edit than to rebuild.
          </p>
        </section>

        <section>
          <h2>Rick Rubin and the creative act</h2>
          <p>
            In <em>The Creative Act: A Way of Being</em>, Rick Rubin writes about creativity as a
            way of noticing and refining. Not inventing from nothing, but paying attention to what
            is already there and making it clearer. That resonates deeply with how I work on
            components. I do not sit down and dream up abstractions. I ship something, then I
            watch. I watch where developers hesitate. I watch where they open the docs. I watch
            where they file bugs that are actually confusion, not defects.
          </p>
          <p>
            Rubin says the work is never just to create. The work is to tune. To listen. To go
            back in and adjust until the thing feels right, until it resonates at the frequency
            it was meant to. That is exactly what maintaining a design system is. You ship v1 and
            it is wrong in ways you could not predict. Then you tune. You adjust a default here,
            rename a prop there, add an event that should have been there from the start. The
            more you tune, the more the system becomes legible to others, the more it starts to
            feel like it was always that way. That effortlessness is earned. Every single time.
          </p>
        </section>

        <section>
          <h2>My background and why simplicity matters</h2>
          <p>
            I grew up poor. There were long stretches without internet, without access to the
            things most developers take for granted. My references were not Stack Overflow threads
            or YouTube tutorials. They were old books. Tolstoy, Dostoevsky, long romances that
            taught me patience and structure. Without constant feeds, without the noise, I learned
            to sit with ideas until they became clear. I did not have the luxury of distraction,
            so I developed a deep intolerance for unnecessary complexity. If an idea needs a
            hundred words to explain, it is probably the wrong idea.
          </p>
          <p>
            That became my approach to components. No rush to add features. No rush to hide
            complexity behind magic abstractions. Let the system evolve naturally. Let the real
            use cases teach you what is missing, instead of guessing and overbuilding. I have
            seen so many systems collapse under the weight of features nobody asked for. I would
            rather ship something small that works than something large that impresses.
          </p>
          <p>
            It is not about minimalism for its own sake. Minimalism can be its own kind of vanity.
            It is about building something that can grow without breaking the people who rely on
            it. Something honest. Something that does not pretend to be more than it is.
          </p>
        </section>

        <section>
          <h2>The Secret, and what I actually believe</h2>
          <p>
            I have to be honest: I do not believe in <em>The Secret</em>. I take almost everything
            in that book with a heavy grain of salt. The law of attraction, the cosmic ordering,
            the idea that thinking about a parking spot will manifest one. No. I do not buy it.
            Life is not that clean. I grew up in a place where people imagined better things
            every single day and still went to bed hungry. Wishing does not fill an empty plate.
          </p>
          <p>
            But here is the thing. Buried inside all that noise, there is something that is almost
            true. Not mystically true. Practically true. The cycle it describes, stripped of the
            magic: <strong>imagine it, believe it, then execute it</strong>. That sequence, taken
            literally and without the woo, is the most honest description of how I build things.
          </p>
          <p>
            Before I write a single line of code, I imagine the developer using the component. I
            see the markup in my head. I picture how it should feel, how the props should read,
            what the default behavior should be. That is the imagination phase. Then I commit to
            it. I believe it is the right approach, not because the universe told me so, but
            because I tested it against empathy, against accessibility, against the constraints
            of the platform. That is belief grounded in craft, not in faith. And then I execute.
            I write the code. I ship it. I watch it break. I tune it.
          </p>
          <p>
            The difference between <em>The Secret</em> and real work is that real work includes
            the breaking part. The tuning part. The part where you sit with your mistake and fix
            it without pretending the universe owed you a smooth ride. Imagine, believe, execute.
            But also: fail, learn, adjust. That is the full cycle. That is what I actually
            believe in.
          </p>
        </section>

        <section>
          <h2>Trying is the work</h2>
          <p>
            The most important part of everything I do is trying. Not succeeding. Trying. I
            know that sounds obvious, but most people skip it. They plan forever, they debate
            architecture in meetings, they draw diagrams, and then they never ship anything
            because they are afraid it will be wrong. Of course it will be wrong. That is the
            point. You try, and trying leads to failing, and failing is the only real path to
            growth. There is no shortcut. There is no amount of reading or planning that
            replaces the moment you ship something and watch it meet reality.
          </p>
          <p>
            Every component I have ever built started as a wrong version. The first draft of
            every API I designed was missing something obvious that I only discovered after
            someone actually used it. That is not a flaw in my process. That is the process.
            The first version exists to teach me what the second version needs. The second
            version exists to reveal what the third version should simplify. Each failure is
            a layer of knowledge that makes the next attempt more honest.
          </p>
          <p>
            Kanye West said: <em>"I am a proud non-reader of books. I like to get information
            from doing stuff."</em> I do read books, I love books, but the core of what he is
            saying hits me hard. Knowledge built from doing is different from knowledge built
            from watching. When you try and fail, the lesson becomes part of your hands, not
            just your head. You remember it differently. You remember the friction, the
            frustration, the exact moment you understood why something did not work. That memory
            is worth more than any documentation.
          </p>
          <p>
            Failing also teaches you what to protect. After you ship a broken default and watch
            fifty developers struggle with it, you never make that mistake again. After you
            over-engineer an API and watch people ignore half the props, you learn what matters
            and what was ego. The lessons are always specific, always earned, and they compound
            over time into something you could never get from a tutorial.
          </p>
        </section>

        <section>
          <h2>Legacy, ego, and the weight of knowledge</h2>
          <p>
            Kanye also said: <em>"If you have the opportunity to play this game of life, you
            need to appreciate every moment."</em> I think about that when I think about legacy.
            Not legacy as in fame or credit. Legacy as in: did the thing I built make someone
            else's work easier? Did it survive after I stopped touching it? Did it teach the
            next person something useful without me having to be in the room?
          </p>
          <p>
            Building a design system is building legacy in the most literal sense. The components
            I write today will outlive my time on the team. Someone will inherit them, extend
            them, maybe rewrite them entirely. And they should be able to do that without needing
            me. If my code requires my presence to be understood, I have failed. The system
            should carry its own knowledge. The names should explain themselves. The structure
            should teach by existing.
          </p>
          <p>
            That requires wiping the ego completely. And I mean completely. Not partially, not
            the polite version where you say "I am open to feedback" but secretly resent every
            suggestion. I mean genuinely letting go of the idea that your solution is precious.
            It is not. Your code is a draft. Your API is a hypothesis. Your architecture is a
            bet that might be wrong. And if someone comes along and makes it better by changing
            everything you wrote, that is a win. Not a loss. A win.
          </p>
          <p>
            Kanye put it another way: <em>"I refuse to accept other people's ideas of happiness
            for me. As if there's a 'one size fits all' standard for happiness."</em> In
            engineering, I read that as a refusal to accept borrowed opinions about what good
            architecture looks like. Every team, every product, every user base is different.
            The ego wants to copy what worked somewhere else and call it best practice. But
            best practice is just someone else's context applied to your problem. It might fit.
            It might not. You have to try, fail, and find your own answer.
          </p>
          <p>
            Knowledge builds in layers. The first layer is technical: how the browser works, how
            events propagate, how the accessibility tree is constructed. The second layer is
            practical: what breaks in production, what confuses real developers, what defaults
            cause the most support tickets. The third layer is the hardest: wisdom. Knowing when
            to add and when to remove. Knowing when to follow a pattern and when to break it.
            Knowing when your opinion is taste and when it is just stubbornness. That third
            layer only comes from trying, failing, and being honest about which is which.
          </p>
          <p>
            The ego wants to build monuments. Taste wants to build tools. A monument exists to
            be admired. A tool exists to be used. I want my components to be tools. I want them
            to disappear into the work of the people who use them. That is the legacy I care
            about: not that someone remembers my name, but that the system kept working long
            after I left.
          </p>
        </section>

        <section>
          <h2>Work is not what HR thinks it is</h2>
          <p>
            There is a corporate version of work that I have never been able to connect with.
            The HR version. The one made of performance reviews, competency matrices, quarterly
            goals phrased in a language that nobody actually speaks. That version of work treats
            people like resources. It measures output in tickets closed, story points delivered,
            ceremonies attended. It confuses presence with contribution and process with progress.
            That is not the real work. It never was.
          </p>
          <p>
            The real work is being with people. Building something together. Sitting next to
            someone and watching how they solve a problem you have been stuck on for days. It is
            the conversation after the meeting, the sketch on a whiteboard that nobody saves but
            everyone remembers. It is sharing what you know without keeping score, and learning
            from someone without making them feel like a teacher. Work is a relationship, not a
            transaction. And the best systems I have ever been part of were built by people who
            trusted each other enough to be honest, not by people who followed a process chart.
          </p>
          <p>
            The corporate lens wants to standardize collaboration. But collaboration is messy
            and human and specific. It is about rhythm, not rules. You learn the rhythm by being
            there, by paying attention, by caring about the person next to you as much as the
            code in front of you. No framework can replace that. No methodology can simulate it.
          </p>
        </section>

        <section>
          <h2>Observation and stealing mindsets</h2>
          <p>
            One of the most important things I have ever done for my own growth is finding people
            to look up to and watching them closely. Not copying their code. Not imitating their
            style. Watching how they think. How they approach a problem. How they break it down.
            What questions they ask before they write a single line. That is the real knowledge
            transfer, and it does not happen in a wiki or a lunch-and-learn. It happens in
            observation.
          </p>
          <p>
            I am constantly asking myself: how did they arrive at that solution? Not what is the
            solution, but what is the structure of thinking that led them there? What did they
            see that I missed? What did they discard that I would have kept? Self-criticism is
            essential here. You have to be willing to look at your own work and admit that
            someone else's approach was better. Not because they are smarter, but because they
            built a different mental framework, and that framework revealed something yours could
            not.
          </p>
          <p>
            The most important thing about creating knowledge is not learning facts. It is
            stealing mindsets. And I mean that with full respect. When I see someone who
            consistently builds clean, durable, accessible components, I do not want their
            code. I want their lens. I want to understand the sequence of decisions that led
            to the result. How do they prioritize? What do they refuse to compromise on? What
            is the first thing they check? What is the last thing they question? That structure
            of thinking is worth more than any library or tool. You can learn a framework in a
            week. A mindset takes years to build, and it is the only thing that transfers
            across every project, every language, every team.
          </p>
          <p>
            Observation also keeps you humble. When you watch someone solve a problem more
            elegantly than you would, it is a reminder that your way is not the only way. That
            is ego dissolving in real time. And the more it dissolves, the more room you have
            to grow. Self-criticism without observation is just anxiety. But self-criticism
            backed by watching how others work becomes a compass. It tells you where you are,
            where they are, and what the distance between those two points is made of.
          </p>
          <p>
            The best engineers I know are all thieves in that sense. They have stolen mindsets
            from people they admire, combined them, refined them, and made something new. That
            is how taste is built. Not from scratch, but from attention, from respect, and from
            the willingness to look at someone else's work and ask: what can I learn from the
            way they see the world?
          </p>
        </section>

        <section>
          <h2>How I evaluate a component</h2>
          <p>
            Before I ship anything, I sit with it. I ask myself a set of questions that have not
            changed much in four years. They are simple questions, but they are ruthless. If a
            component cannot survive them, it goes back.
          </p>
          <ol>
            <li>Can a developer understand how to use it just by reading the HTML?</li>
            <li>Are native events preserved, or at least intentionally re-exposed?</li>
            <li>Is the default behavior useful without any configuration?</li>
            <li>Does the API stay small but genuinely extensible?</li>
            <li>Can I remove something and make it better?</li>
            <li>Does it work for someone using only a keyboard?</li>
            <li>Does it announce itself correctly to a screen reader?</li>
            <li>Would I want to use this myself at 11pm on a deadline?</li>
          </ol>
          <p>
            That last one is the real test. Not whether it is technically correct, but whether it
            respects the person on the other side. If I would curse at it, someone else will too.
          </p>

          <blockquote brixsBlockquote="warning">
            <span brixsTag="warning">Reminder:</span> Complexity is easy to add and hard to remove.
            Every prop you add is a promise you have to keep forever.
          </blockquote>
        </section>

        <footer>
          <h2>Conclusion</h2>
          <p>
            Taste is the architecture behind a component. It is empathy turned into code. It is
            restraint when you could add more but know you should not. It is the courage to make
            small, precise changes instead of grand rewrites. It is the willingness to stay open,
            to keep APIs honest, and to build for the developer who will live inside the system
            long after you have moved on to the next thing.
          </p>
          <p>
            I want to build things that feel like they were always there. Things that do not need
            a conference talk to explain. Things that respect the platform, respect the user, and
            respect the developer's time. Optimized, open, accessible, and never complicated.
            Things born from trying, shaped by failing, and finished by letting go of the ego
            that wanted them to be impressive instead of useful.
          </p>
          <p>
            When Lex Fridman asked Kanye what he hopes his legacy is, he said: <em>"To be
            forgotten. Because the memory, there's ego in memory, in the memories. Who designed
            the sidewalk? Who designed the water fountain? Who designed the stop sign? Who
            designed the stoplight? These things are so ubiquitous that the person that designed
            them is forgotten. If it's a good idea, it's a God idea, and no VCs can own it."</em>
          </p>
          <p>
            That is the most honest thing I have ever heard anyone say about design. The sidewalk
            does not have a signature. The stoplight does not credit its author. They just work.
            They serve billions of people every single day without anyone asking who made them.
            That is the highest form of design: something so right, so inevitable, that it
            becomes invisible. The ego wants a name on the building. Taste wants the building
            to stand on its own.
          </p>
          <p>
            That is what I want for my components. I want them to be forgotten. I want a
            developer to use them at 11pm and never once think about who built them. I want the
            API to feel so obvious that it could not have been any other way. I want the
            defaults to be so right that nobody notices they are there. Every feature I chose
            not to add, every abstraction I refused to build, every shortcut I did not take,
            made the system what it is. The restraint is the architecture. The silence is the
            rhythm. The failure is the foundation. And the best possible outcome is that none
            of it is remembered, because all of it just works.
          </p>
          <p>
            That is how I want to build. That is all I want to build.
          </p>
        </footer>
      </article>
    </div>

  </div>
</section>
