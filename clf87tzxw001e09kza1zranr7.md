---
title: "A Frontend Survival Guide: CSS Flexbox - #1"
datePublished: Tue Mar 14 2023 12:14:27 GMT+0000 (Coordinated Universal Time)
cuid: clf87tzxw001e09kza1zranr7
slug: a-frontend-survival-guide-css-flexbox-1
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678795352492/b286222c-0673-483c-9174-7456010cb1a9.png
tags: flexbox, css3, css, css-flexbox, frontend

---

There are a million tutorials online that tell you about all the CSS Flexbox properties, how to use them etc., and I for once do not want to be among them. This is not an article that will throw jargon at you, but rather explain CSS Flexbox in complete beginner terms.

This will purely be about what and why CSS flexbox is and what're the most commonly used properties plus some resources for further study.

### What and Why is Flexbox even?

Flexbox is just short for the flexible box module. It provides a clean way to arrange items in a container as either columns or rows. Flexbox is used to create 2D layouts which most of us would need in 99% of cases, for 3D and complex layouts we would need CSS Grid (which we will cover in a later article).

1. Flexbox was created so that we can give our container the ability to alter its items' height, width, and order to best fill the available space since all of us are on different devices and screen sizes. There are quite a handful of container properties and item properties that work together to prevent CSS overflow or underflow. This makes the core of responsive web development.
    
2. Flexbox is direction-agnostic, that's just a fancy of saying that flexbox is free from directional constraints, meaning we can put our elements in either row or column. This is important as our usual layouts such as `inline` or `block` just are not equipped to handle the increasingly difficult requirements of complex web applications.
    

### Terminology

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678791991496/68add9e2-31d5-486f-a546-0f6c44dbd095.jpeg align="center")

No need to think too heavily about it, just understand that there are two axes: Main and Cross, in which we work, the parent element is known as the "flex container" and the child element is known as the "flex item".

### The only properties you will need (probably)

* **display**
    
    We use this to define a flex, inline or block container.
    
    ```css
    .container {
        display: flex; /* enables flex context to its direct children*/
    }
    ```
    
* **flex-direction**
    
    This is used to establish the main axis, basically defining the direction of flex items in the container.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678794282597/c1bf2bcc-b856-4605-9ae6-56e1a3f74785.png align="center")
    
    ```css
    .container {
      flex-direction: row | row-reverse | column | column-reverse;
    }
    ```
    
* **justify-content**
    
    Helps with the distribution of extra space and defines the alignment along the main axis defined by flex-direction.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678794579011/60198a31-8e68-4a0d-8016-f74ea6b958a8.png align="center")
    
    ```css
    .container {
      justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
    }
    ```
    
* **align-items**
    
    Think of it as the justify-content version for the cross-axis.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678794793110/be4a98d2-cdb6-41b5-88fb-176385711d82.png align="center")
    

```css
.container {
  align-items: stretch | flex-start | flex-end | center | baseline;
}
```

### Conclusion

The above are the only flexbox properties you will use most of the time. You might also have to use `gap` and/or `flex` but they are relatively less used, and you can always look at the million tutorials available online. The idea for this article to help you understand flexbox at its basic and what is it that it does. Once you have the basics down, you can use flexbox for advanced stuff as well.

Below are resources that will help you learn flexbox in detail.

1. [Overview](https://youtu.be/K74l26pE4YA)
    
2. [A fun game to help you understand flexbox by doing](https://flexboxfroggy.com/)
    
3. [For those addicted to tutorial hell](https://youtu.be/u044iM9xsWU)
    
4. [Pure Documentation](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)
    
5. [For Visual Learners](https://poonia.github.io/flexbox/)
    

Thanks!