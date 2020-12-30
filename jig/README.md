Even though I was succesful with flashing using this design, I'm not happy with it. I think it can be made into a single part and simplyfied for more reliable printing. \
If there is enough interest I'll rework it.

# Printable jig
![1](/pictures/3.jpg)
![1](/pictures/1.jpg)

This is a 3d printable jig for flashing SOIC-16 bios chip on Thinkpad X200 using jumper cables.\
It's made of 2 parts: base (aligns everything) and springs (keeps pins under tension).

### Print settings
Printed with Prusa MK3 0.20 "quality" preset (extrusion width = 0.45), generic PLA.\
2 perimeters, no solid top/bottom layers. Infill is up to you.

## How to use
Before installing make sure jumper wires move freely in their "wells".\
Install base first. It takes screws you just pulled from your X200.\
Insert jumper wires.\
Install springs.\
You can pull on wires to make sure they spring back.

### Wiring
After installing the jig jumper wires will have following layout (top view):

|      |      |
| ---- | ---- |
|      | MISO |
| GND  | CS   |
| MOSI | VCC  |
| CLK  |      |
