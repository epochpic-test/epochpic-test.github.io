+++
title = ""
draft = false  # Is this a draft? true/false
toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.

# Add menu entry to sidebar.
linktitle = "Deck example"
[menu.tutorial]
  parent = "Content"
  weight = 160
+++

```perl
begin:control
  nx = 200

  # Size of domain
  x_min = -4 * micron
  x_max = -x_min

  # Final time of simulation
  t_end = 50 * femto

  #stdout_frequency = 10
end:control


begin:boundaries
  bc_x_min = open
  #bc_x_min = simple_laser
  bc_x_max = open
end:boundaries


begin:laser
  boundary = x_min
  intensity_w_cm2 = 1.0e15
  lambda = 1 * micron
  phase = pi / 2
  t_profile = gauss(time, 2*micron/c, 1*micron/c)
  t_end = 4 * micron / c
end:laser


begin:output
  dt_snapshot = 1 * micron / c

  # Properties on grid
  grid = always
  ey = always
end:output
```
