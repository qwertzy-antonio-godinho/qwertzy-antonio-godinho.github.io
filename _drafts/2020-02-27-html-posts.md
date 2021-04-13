---
layout: post
title: Or maybe Markdown
subtitle: This post explains how you can write posts using Markdown.
tags: [guide, markdown]
---

<span class="color-red">[NEW]:</span> Now you can also create a private post, which will not be visible on the blog 

Posts should be named as `%yyyy-%mm-%dd-your-post-title-here.md`, and placed in the `_posts/` directory. Drafts can be kept in `_drafts/` directory.

-------------

# This is a heading
## This is a sub-heading
### This is a sub-sub-heading

<span class="color-blue">Some</span>
<span class="color-green">cool</span>
<span class="color-orange">colorful</span>
<span class="color-red">text.</span><br>

<span class="highlight-blue">And</span>
<span class="highlight-green">some</span>
<span class="highlight-orange">highlighting</span>
<span class="highlight-red">styles.</span>

**Here is a bulleted list,**
 - This is a bullet point
 - This is another bullet point


**Here is a numbered list,**
1. You can also number things down.
2. And so on.

**Here is a sample code snippet in C,**
```C
#include <stdio.h>

int main(void){
    printf("hello world!");
}
```

**Here is a horizontal rule,**

--------------

**Here is a blockquote,**

> There is no such thing as a hopeless situation. Every single 
> circumstances of your life can change!

**Here is a table,**

ID  | Name   | Subject
----|--------|--------
201 | John   | Physics
202 | Doe    | Chemistry
203 | Samson | Biology

**Here is a link,**<br>
[GitHub Inc.](https://github.com) is a web-based hosting service
for version control using Git

**Here is an image,**<br>
![](../assets/autumn.jpg)

# Default tint3 configuration file.
#
# For information on manually configuring tint3 see:
#   https://github.com/jmc-88/tint3

# {{{ backgrounds
# ID 1
rounded = 6
border_width = 1
background_color = #000000 0
border_color = #e1e1e1 25

# ID 2
rounded = 3
border_width = 0
background_color = #000000 0
background_color_hover = #7f7f7f 40
background_color_pressed = #7f7f7f 70
border_color = #000000 50

# ID 3
rounded = 6
border_width = 1
background_color = #000000 40
border_color = #ffffff 60

# ID 4
rounded = 3
border_width = 1
background_color = #3f3f3f 90
border_color = #ffffff 90
# }}} backgrounds

# {{{ panel
panel_monitor = all
panel_position =  bottom center horizontal
panel_size = 100% 40
panel_margin = 0 0
panel_padding = 0 0 0
panel_dock = 0
wm_menu = 1
panel_layer = normal
panel_background_id = 2
# }}} panel

# {{{ taskbar
taskbar_mode = multi_desktop
taskbar_padding = 0 0 5
taskbar_background_id = 0
taskbar_active_background_id = 0
# }}} taskbar

# {{{ tasks
urgent_nb_of_blink = 20
task_icon = 1
task_text = 0
task_centered = 1
task_maximum_size = 48 30
task_padding = 9 8 0
task_background_id = 2
task_active_background_id = 3
task_urgent_background_id = 2
task_iconified_background_id = 2
# }}} tasks

# {{{ task icons
task_icon_asb = 100 0 0
task_active_icon_asb = 100 0 0
task_urgent_icon_asb = 100 0 0
task_iconified_icon_asb = 100 0 0
# }}} task icons

# {{{ system tray
systray = 1
systray_padding = 0 2 6
systray_sort = left2right
systray_background_id = 0
systray_icon_size = 22
systray_icon_asb = 100 0 0
# }}} system tray

# {{{ tooltips
tooltip = 1
tooltip_padding = 0 0
tooltip_show_timeout = 0.0
tooltip_hide_timeout = 0.0
tooltip_background_id = 0
tooltip_font = Mononoki Regular 11
tooltip_font_color = #dcdcdc 70
# }}} tooltips

# {{{ mouse
mouse_middle = none
mouse_right = none
mouse_scroll_up = toggle
mouse_scroll_down = iconify
mouse_effects = 0
mouse_hover_icon_asb = 100 0 25
mouse_pressed_icon_asb = 100 0 -25
# }}} mouse