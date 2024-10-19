
## 1. Always use `position: absolute`

`position: absolute` means that the box is placed based on its parent box. It uses the top, left, right, and bottom values to decide where it goes within that parent. 

We often think that a box inside another box will be positioned according to where it is inside the parent box. This setting makes it look like that. However, we can also move the child box outside of the parent box if we want.

When we set `position: absolute` for the child box, it calculates its position from the top-left corner of its parent. If we want to place the child outside the parent, we can use negative values like this:

- `top: -10px;` means the top of the child box will be 10 pixels above the top of the parent box.
- `left: -10px;` means the left side of the child box will be 10 pixels to the left of the parent box.

In this context, "parent" refers to any element that has a position set to `absolute` or `relative`, not `static`.

In this context, "box" and "element" mean the same thing. You can assume them to be a div.

## 2. For CSS layouts, there are only two factors, how much space will the box take and where will that box be placed.

We can control position using `position`. Size of the box can be controlled by `height` and `width`.

## 3. Always use `box-sizing: border-box`

A box is made up of 4 things: 
1. **content area**: area where your text is shown.
2. **padding area**: distance between the border and your content area.
3. **border area**: width of the border of the box.
4. **margin**: distance between other boxes surrounding the current box. 

When you use border-box, the `width` and `height` you set include the content, padding, and borders. So, if you set a box to be 200 pixels wide with border-box, it will remain 200 pixels wide even if you add padding or borders. The browser automatically adjusts the content area to fit within this total size.

 You can ofcourse, revert to the default `box-sizing` property but only if you want your `height` and `width` to affect the size of the content area of the box and not the complete box.


## 4. So, use below default in your css file.

```
* {
  box-sizing: border-box;
  position: absolute;
}
```

If you think there is a design which will cause problems when you use these then don't use it. But this makes making layouts easier.
