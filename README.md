# Gridfinity Rebuilt in OpenSCAD 

A ground-up port (with a few extra features) of the stock [gridfinity](https://www.youtube.com/watch?v=ra_9zU-mnl8) bins in OpenSCAD. Open to feedback, because I could not feasibly test all combinations of bins. I tried my best to exactly match the original gridfinity dimensions, but some of the geometry is slightly incorrect (mainly fillets). However, I think they are negligible differences, and will not appear in the printed model.

[<img src="./images/base_dimension.gif" width="320">]()
[<img src="./images/compartment_dimension.gif" width="320">]()
[<img src="./images/height_dimension.gif" width="320">]()
[<img src="./images/tab_dimension.gif" width="320">]()
[<img src="./images/holes_dimension.gif" width="320">]()
[<img src="./images/custom_dimension.gif" width="320">]()

## Features
- any size of bin (width/length/height)
- height by units, internal depth, or overall size
- any number of compartments (along both X and Y axis)
- togglable scoop
- togglable tabs, split tabs, and tab alignment
- togglable holes (with togglable supportless printing hole structures)
- manual compartment construction (make the most wacky bins imaginable)
- togglable lip (if you don't care for stackability)

### Printable Holes
The printable holes allow your slicer to bridge the gap inside the countersunk magnet hole (using the technique shown [here](https://www.youtube.com/watch?v=W8FbHTcB05w)) so that supports are not needed.

[<img src="./images/slicer_holes.png" height="200">]()
[<img src="./images/slicer_holes_top.png" height="200">]()

## Recommendations
For best results, use a version of OpenSCAD with the fast-csg feature. As of writing, this feature is only implemented in the [development snapshots](https://openscad.org/downloads.html). To enable the feature, go to Edit > Preferences > Features > fast-csg. On my computer, this sped up rendering from 10 minutes down to a couple of seconds, even for comically large bins.  

## Global Parameters
These are the parameters that define the type of bin being constructed. Therefore, they impact everything that does not relate to compartments, such as the base and the outside wall. 

Parameter | Range | Description
--- | ----- | ---
`gridx` | {n>0\|n∈Z} | number of bases along the x-axis  
`gridy` | {n>0\|n∈Z} | number of bases along the y-axis  
`length` | {n>0\|n∈R} | size of the square bases in millimeters, default 42  
`gridz` | {n>0\|n∈R} | height value (can be interpreted in various ways)  
`gridz_define` | {0,1,2} | • (0) Unit height: `gridz` is the height in units (7mm increments). The stock bins have a unit height value of 2, 3, or 6. The overall height of the bin is `7*u + 3.8` millimeters, where the 3.8 is the height of the top lip. <br>     • (1) Internal height: `gridz` is the height from the bottom of a compartment to the top of the tab in millimeters. This is effectively the maximum height of an object that can fit inside the bin. <br>     • (2) External height: `gridz` is the overall height of the bin, from the base to the upper lip fillet. This is effectively how deep a drawer must be to fit the bin. 
`enable_holes` | boolean | toggle holes for magnets and M3 screws on the base of the bin
`enable_hole_slit` | boolean | toggle the printable countersunk hole cut
`enable_zsnap` | boolean | have the bin increase in height until it matches the nearest 7mm increment. Useful if a bin needs to be a minimum internal size to fit a part (i.e. `gridz_define == 1`) but you still want the bin to fit within the sizing standards of the gridfinity system
`enable_lip` | boolean | toggles the lip at the top of the bin that allows for bin stacking

## Modules  
Run these functions inside the *Commands* section of *gridfinity-rebuilt.scad*.

### `gridfinityEqual(n_divx, n_divy, style_tab, enable_scoop)`  
Generates the "traditional" bins. It is a utility function that creates evenly distributed compartments. 

Parameter | Range | Description
--- | ----- | ---
`n_divx` | {n>0\|n∈Z}  | number of compartments along X
`n_divy` | {n>0\|n∈Z}  | number of compartments along Y
`style_tab` | {0,1,2,3,4,5} | how the tabs for labels are generated. <br>     • (0) Full tabs across the entire compartment <br>     • (1) automatic tabs, meaning left aligned tabs on the left edge, right aligned tabs on right edge, center tabs otherwise <br>     • (2) left aligned tabs <br>     • (3) center aligned tabs <br>     • (4) right aligned tabs <br>     • (5) no tabs
`enable_scoop` | boolean | toggles the scoopy bit on the bottom edge that allows easy removal of items

```
// Example: this generates 9 compartments in a 3x3 grid, and all compartments have a full tab and a scoop
gridfinityEqual(n_divx = 3, n_divy = 3, style_tab = 0, enable_scoop = true);
```

### `gridfinityCustom() {}`

If you want to get crazy with it, you can take control of your destiny and manually place the compartments. This can be done using this module, which will cut all child objects into the container. There are various modules that are exposed for this purpose.  

### `cut(x, y, w, h, t, s)` 
Cuts a single compartment into the bin at the provided location with the provided attributes. The coordinate system for compartments originates (0,0) at the bottom left corner of the bin, where 1 unit is the length of 1 base. Positive X and positive Y are in the same direction as the global coordinate system.
Parameter | Range | Description
--- | ----- | ---
`x` | {n>=0\|n∈R} | X coordinate of the compartment (position of left edge of compartment)
`y` | {n>=0\|n∈R} | Y coordinate of the compartment (position of bottom edge of compartment)
`w` | {n>0\|n∈R} | Width of the compartment, in base units (1 unit = 1 `length`)
`h` | {n>0\|n∈R} | Height of the compartment, in base units (1 unit = 1 `length`)
`t` | {0,1,2,3,4,5} | how the tabs for labels are generated for this specfic compartment. <br>     • (0) Full tabs across the entire compartment <br>     • (1) automatic tabs, meaning left aligned tabs on the left edge, right aligned tabs on right edge, center tabs otherwise <br>     • (2) left aligned tabs <br>     • (3) center aligned tabs <br>     • (4) right aligned tabs <br>     • (5) no tabs
`s` | boolean | toggles the scoopy bit on the bottom edge that allows easy removal of items, for this specific compartment

```
// Example: Assuming a gridx of 3 and a gridy of 3
// this cuts two compartments that are both 1 wide and 2 high. 
// One is on the bottom left, and the other is at the top right. 
gridfinityCustom() {
    cut(0, 0, 1, 2, 0, true);
    cut(2, 1, 1, 2, 0, true);
}
```

### `cut_move(x, y, w, h)` 
Moves all of its children from the global origin to the center of the area that a compartment would normally fill, and uses them to cut from the bin. This allows you to easily make custom cutouts in the bin.
Parameter | Range | Description
--- | ----- | ---
`x` | {n>=0\|n∈R} | X coordinate of the area (position of left edge)
`y` | {n>=0\|n∈R} | Y coordinate of the area (position of bottom edge)
`w` | {n>0\|n∈R} | Width of the area, in base units (1 unit = 1 `length`)
`h` | {n>0\|n∈R} | Height of the area, in base units (1 unit = 1 `length`)

```
// Example: assuming gridx of 3 and gridy of 3
// cuts a cylindrical hole of radius 5
// hole center is located 1/2 units from the right edge of the bin, and 1 unit from the top
gridfinityCustom() {
    cut_move(x=2, y=1, w=1, h=2) {
          cylinder(r=5, h=100, center=true);
    }
}
```

More complex examples of all modules can be found in the scripts.

## Enjoy!

[<img src="./images/spin.gif" width="160">]()

[Gridfinity](https://www.youtube.com/watch?v=ra_9zU-mnl8) by [Zack Freedman](https://www.youtube.com/c/ZackFreedman/about)
