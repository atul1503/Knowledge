
## 1. Always use `position: absolute`

`position: absolute` means that the box is placed based on its parent element. It uses the top, left, right, and bottom values to decide where it goes within that parent. 

We often think that a box inside another box will be positioned according to where it is inside the parent box. This setting makes it look like that. However, we can also move the child box outside of the parent box if we want.

When we set `position: absolute` for the child box, it calculates its position from the top-left corner of its parent. If we want to place the child outside the parent, we can use negative values like this:

- `top: -10px;` means the top of the child box will be 10 pixels above the top of the parent box.
- `left: -10px;` means the left side of the child box will be 10 pixels to the left of the parent box.

In this context, "parent" refers to any element that has a position set to `absolute` or `relative`, not `static`.

## 2. For CSS layouts, there are only two factors, how much space will the box take and where will that box be placed.

We can control position using `position`. Size of the box can be controlled by `height` and `width`. 


