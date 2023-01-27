# reveal animate


Reveal animation is a big standard for SPA website.

You can find plenty of "full" libraries to add this functionality to your website. 

Now with the help of IntersectionObserver and a nice CSS animation library, you can do that easily. Big advantage, you are in control!

You can find animate library here:

https://animate.style/

By the way, if you need only 2 or 3 animations (left, right zoom), you can just add those needed (see at the end), that'll save some loading time. 


---
*use ctrl-click or right button on link, open in new tab*


[Test it on CodePen](https://codepen.io/pierfarrugia/pen/jOKLmzG)

![alt text](https://aonecommunication.ch/content/text_overflow/text_overflow.webp)

[See it in action full screen](https://aonecommunication.ch/content/text_overflow/text_overflow.html)

[GitHub](https://github.com/pierfarrugia/text_overflow)


---
## HTML Structure

HTML structure is the following:

```HTML
<section id="home"></section>

<section id="section1">
    <div class="row">
        <div class="c1-2 animate__animated" data-reveal="animate__backInLeft">

        </div>
        <div class="c1-2 animate__animated animate__delay-2s" data-reveal="animate__backInRight">

        </div>
    </div>
</section>

<section id="section2">
    <div class="row gap-md">
        <div class="c1-3 animate__animated" data-reveal="animate__bounceInLeft">

        </div>
        <div class="c1-3 animate__animated animate__delay-2s" data-reveal="animate__bounceInDown">

        </div>
        <div class="c1-3 animate__animated animate__delay-4s" data-reveal="animate__bounceInRight">

        </div>
    </div>
</section>

<div style="margin-top: 7rem;"></div>
```

- section home is to have some space before to see the reveal
- 2 sections: first with 2 columns, second with 3 columns
- last div is to have some space at the end for the reveal

For each element to reveal with animation, there is some parameters:
- animate__animated: that's the class for animate.css
- with 2 elements to animate, class animate__delay-2s is added on the second. to have a little delay between left and right. With 3 elements: delay 2s second and 4s third (del)
- data-reveal attribute is the exact name of the animation you want. Instead of putting the class animation from animate.css in the class, you put it in data attribute data-reveal


*animate.css utility classes have 4 pre-defined delay: animate__delay-2s, animate__delay-3s, animate__delay-4s, animate__delay-5s*

*if you want more delay options, you can edit the animate.css in editor*



---
## CSS

CSS for the demo:

```css
        section {
    position: relative;
    width: 100%;
    min-height: 80vh;
}

#home {
    position: relative;
    width: 100%;
    min-height: 100vh;
}

section > div > div {
    background-color: red;
    width: 100%;
    min-height: 60vh;
}

.row {
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    grid-template-rows: 1fr auto;
    gap: 10px;
}

.c1-2 {
    grid-column: auto / span 6;
}

.c1-3 {
    grid-column: auto / span 4;
}
```


---
## Javascript

### init

The first needed function is to init our reveal:
```javascript
const initOberverReveal = () => {
    let elsAnime = document.querySelectorAll('[data-reveal]');
    elsAnime.forEach((el) => {
        el.style.opacity = '0';
        observerAnime.observe(el);
    });
}
window.addEventListener('load', initOberverReveal);
```

We select all elements with [data-reveal].

For each we put the opacity to 0, we have to hide element before it is revealed.

Finally, we call the function to observe all the data-reveal elements.

### observe and reveal

```javascript
const observerAnime = new IntersectionObserver(function (entries, self) {
    //console.log(entries);
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            let el = entry.target;
            el.style.opacity = '1';
            el.classList.add(el.dataset.reveal);
            self.unobserve(entry.target);
        }
    });
}, {
    root: null,
    threshold: [1],
    rootMargin: '0px 0px 0px 0px'
});
```

Last part contains the default parameters for IntersectionObserver. It's here if you want to modify something (otherwise you can delete it). You can check all parameters here:

https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API


For all the "entries" (data-reveal elements), we check if it's intersecting. If it's intersecting:
- we put opacity to 1
- for this element we add in class the data-reveal attribute value: animation will fire now
- last line is saying to stop observing now: meaning this element won't fire intersection anymore. If you don't put this line, animation will play always when scrolling down but also up. With this line it'll play once only.

It's working, but we can refine it a little!


### refine with reset

Animations play once, and element is no more checks by intersectionobserver.

Now what about, we play the animation once going down (and up) the document, but we reset the animation when we go back top of document?

To check if we are back top of document the easiest way is to check window scroll:
```javascript
window.addEventListener("scroll", function(){
    if(window.scrollY === 0) {
        // top of the page reach
    } else {
        //...
    }
});
```
That works well, but why using intersectionObserver for the reveal to avoid to exhaust browser with continuous scroll checking if it's now to use it just to check 0!!!

We are going to add a ghost div on top of page (a sentinel) and use intersectionObserver again to check this element.

Add a div before home section
```HTML
<div id="topOfPage"></div>

<section id="home"></section>

...
```

This div has the following CSS:

```css
#topOfPage {
    opacity: 0;
    height: 0.01px;
}
```
It will have full width, opacity 0, and height of 0.01px, we don't see it but it's real in the DOM.


Now we'll check it intersecting with intersectionObserver, this time no need to unobserve, lot simpler one

Final javascript is: 

```javascript
    /* GO TOP ON LOAD */
window.onload = function () {
    if (history.scrollRestoration) {
        history.scrollRestoration = 'manual';
    } else {
        window.onbeforeunload = function () {
            window.scrollTo(0, 0);
        };
    }
}

/* ============== observe top of page ============== */
const observerTopOfPage = new IntersectionObserver(function (entries, self) {
    //console.log(entries);
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            let el = entry.target;
            initOberverReveal();
            console.log(el);
        }
    });
}, {
    root: null,
    threshold: [1],
    rootMargin: '0px 0px 0px 0px'
});

/* ============== init top of page ============== */
const initOberverTopOfPage = () => {
    let elTop = document.querySelector('#topOfPage');
    observerTopOfPage.observe(elTop);
}
window.addEventListener('load', initOberverTopOfPage);


/*
 * INTERSECTION OBSERVER ANIMATION
 */

/* ============== reveal animation: observe ============== */
const observerAnime = new IntersectionObserver(function (entries, self) {
    //console.log(entries);
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            let el = entry.target;
            el.style.opacity = '1';

            el.classList.add(el.dataset.reveal);
            console.log(el);
            self.unobserve(entry.target);
        }
    });
}, {
    root: null,
    threshold: [1],
    rootMargin: '0px 0px 0px 0px'
});

/* ============== reveal animation: init ============== */
const initOberverReveal = () => {
    let elsAnime = document.querySelectorAll('[data-reveal]');
    elsAnime.forEach((el) => {
        el.style.opacity = '0';
        el.classList.remove(el.dataset.reveal);
        observerAnime.observe(el);
    });
}
window.addEventListener('load', initOberverReveal);

```





Try the demo, to see it working.

Javascript here is deliberately detailed. Not compression, or reduction of 2-3 lines in one to be more understandable. You can easily reduce it.

There would be another easy "opening" option using a lightbox, but personnaly I don't like so much opening modal windows when not necessary.

Hope that can help.

Thanks for reading

*(tested on Chrome, Firefox, Edge, Safari)*
