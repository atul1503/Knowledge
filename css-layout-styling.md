
## Always use `position: absolute`

`position: absolute` means that the box will be placed according to the position of its parent element. The box uses the values for top, left, right, and bottom to determine where it should be positioned within that parent.
So because we are in the habit of thinking that the box inside another box will be positioned in the parent box according to the position inside parent box, this solution makes it appear like that. But still we have freedom to make the child box go outside the parent box. This is because when `position: absolute` is set for the child, the child uses top,left, right and bottom values calculated from the top left corner of its parent. So if we want to put child outside of the parent we can do something like this: 
```
top: -10px;
left: -10px;
```

`top: -10px` will ensure that the top side of the child box is -10 pixels from the top side of the parent box towards the downward direction, which means it is 10 pixels above of the top side of the parent box. Similarly, `left: -10px` will ensure that the left side of the child box will be -10 pixels left of the left side of the parent box towards the right direction, which means the left side of the child box will be 10 pixels left of parent's left side.

**Here parent means the parent which is either having `position` as `absoute` or `relative` and not `static`.**


## For CSS layouts, there are only two factors, how much space will the box take and where will that box be placed.

We can control position using `position`. Size of the box can be controlled by `height` and `width`. 


