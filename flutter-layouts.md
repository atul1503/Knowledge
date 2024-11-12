## How to create flex layout in flutter

Flex container is created by `Flex` widget. Flex child is created by `Flexible` widget.
`Flex` has `children:` argument to specify its children.
`Flexible` has two arguments that we need to set:
1. `fit:` has to be set to either `FlexFit.loose` or `FlexFit.tight`. Tight means that it will be forced to follow the given constraints. Loose means that its constraint is present but it has to adhere to it only if required.
2. `flex:` has to be an integer value. Its like a proportion value. For eg, if Flex container has x space and it has two children with their `flex` set to 4 and 3 respectively then space will be divided as (4x/7) and (3x/7). This is the constraint that is put on the children. If the fit is loose then children can take upto 4x/7 and 3x/7 space(in this case) but if fit is tight then children have to take exactly 4x/7 and 3x/7 space.

**Flex,like this, is best for layouts where you don't know the exact postion they have to be in.

## How to create absolute positioning.

In layouts, this is only required if you want to set the exact position of children in a parent.
Use `Stack` as parent and `Positioned` as child. 
`Positioned` has `top,left,bottom,right` properties to set its position inside the `Stack` parent. Any of its children cannot display itself out of the stack widget.
If the child's position is set in such a way that it is supposed to show outside the stack parent, then the child doesnot even render on the screen. But if the stack is unbounded then the child may display itself without any problem no matter what you set its position.


## Thumb rules
1. Using stack and postioned, you can create any layout you want, but for standard layouts, always use flex and flexible.
2. Use `Padding` as parent of the child to which you are trying to give padding.
3. If you want to set the position and size of a `Container` to certain size and if the sizing of the container is taking up the full space of the container then replace container with a stack and use positioned as one of the children of the stack and then put that container as the child of that positioned. Something like this.

   ```
     Stack
        Positioned
           Container //Your Container that you are trying to have a certain size.
   ```
   With `Positioned` here, you can set both the size(heigth and width) and position(left,right,top,bottom) of the container.

