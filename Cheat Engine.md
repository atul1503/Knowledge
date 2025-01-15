## How to do pointer scanning to find pointers to your addresses
Pointer maps are files that cheat engine generates that helps in pointer scanning. You have to generate more than one pointer maps to help with pointer scans. Pointer maps are actually representation of all the pointers who are pointing to an address. Pointer maps can be stored in files for later use. During pointer scans, you can select cheat engine to refer to these pointer maps and also the address which was used to generate the pointer map, to find pointers to your target address.

Pointer scan is a process in cheat engine that uses the pointer maps and the address from which you created those pointer map to find pointers that always point to your address even if you restart the game.


For eg, If you want to know the pointers to the health of the player. These are the steps: 
* First find it by using memory scan or using instructions. Name this address as `Health 1`
* Generate a pointer map using this address and save it with  suitable name. Save the pointer map as `Health 1` in local disk.
* Close the game or let the game generate a new address for your health. Name this address as `Health 2`.
* Again create a pointer map with this new health address and save it on disk as `Health 2`.
  Now you have created two pointer maps and their associate address (here, Health 1 and Health 2).
* Close the game again or allow the health address to change and name it as `Health 3`
* Now, generate a pointer map for this address also and name this pointer map as `Health 3`.
* Right click on `Health 3` address and select `pointer scan for this address`.
  A new window will open up, called `Pointer scan options` window.
* In that window, click on  `use saved pointermap` to select the pointer map that you created with `Health 3`.
  This is required so that the pointer scan can know which pointers are pointing to `Health 3`
* Now click on `compare results with saved pointermaps`.
  A multi file attachment box will appear.
  Click on each attachment button and attach all the pointer maps generated (here, attach `Health 1`,`Health 2`). But don't attach `Health 3` pointer map.
  Beside each attachment you will see a text box. You have to enter the associated addresses that were generated with each of the pointermaps that you attached.
* Since we have already named those address with the same name as the pointer maps, select that address from the dropdown of the text box. Foe eg, beside the `Health 1` attachment, select `Health 1` address from the dropdown. Do this for each attachment textboxes. Select the respective addresses. The address are shown here because those addresses are in the address list table.
* Now, check the `pointers must end with specific offsets` option.
  When you select the option, a text box will appear where you have to specify the offset of your address from its base. This you can find out by doing `what access this address` on any of the `Health` address. The offset can be shown in something like `[eax+50]` where `50` is the offset. There can be multiple offsets also. There can zero offset also.
* Whatever the offset you like that you found from the `what accessses` instructions, you can enter in the text box.
* Finally click on `OK`.
  Cheat engine will try to find pointers for your address using those pointer maps. Finally when found, if you double click on any row, that pointer will be added to your address list. I would recommend adding multiple pointers to the address list and checking each one with different scenarios.

Many pointers can do the job. Pointers are different from addresses. There may be only one health address for your player but there can be multiple pointers pointing to that health address. 


**Also during the whole process, if you have to close the game to allow game to generate new address for your health then don't close cheat engine because even though the pointer maps are stored in the disk, your associated address like `Health 1` etc are still stored in the table. You will need those addresses for pointer scanning**  


## General tips

* In lua, never put long running for loops in hotkey or timers. If you want to run something that long, use timers. But just for efficiency make sure that you disable timers when not required.
* Also in case of get memory record by description, the value of the record can be a non numeric also sometimes so make sure that if `tonumber(x.Value)` gives `nil` then don't move ahead with operations. Here, `x` is the memory record object.
* Also make sure that before referencing a memory record, make sure that it exists in the table.
