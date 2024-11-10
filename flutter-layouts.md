## How to create flex layout in flutter

Flex container is created by `Flex` widget. Flex child is created by `Flexible` widget.
`Flex` has `children:` argument to specify its children.
`Flexible` has two arguments that we need to set:
1. `fit:` has to be set to either `FlexFit.loose` or `FlexFit.tight`. Tight means that it will be forced to follow the given constraints. Loose means that its constraint is present but it has to adhere to it only if required.
2. `flex:` has to be an integer value. Its like a proportion value. For eg, if Flex container has x space and it has two children with their `flex` set to 4 and 3 respectively then space will be divided as (4x/7) and (3x/7). This is the constraint that is put on the children. If the fit is loose then children can take upto 4x/7 and 3x/7 space(in this case) but if fit is tight then children have to take exactly 4x/7 and 3x/7 space.

**Flex,like this, is best for layouts where you don't know the exact postion they have to be in.

## How to create absolute positioning.

In layouts, this is only required if you want to set the exact position of children in a container.
Use `Stack` as container and `Positioned` as child. 
`Positioned` has `top,left,bottom,right` properties to set its position inside the `Stack` container. Any of its children cannot get out of the stack container.
