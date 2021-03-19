---
layout: post
title: "Palette Swapping With Shaders"
date: 2019-10-06
author: pierre
tab: blog
tags: vagabond pcg
---
Hi everyone!

In this devlog, I will show you a technique that I love and that I will abuse in [Vagabond](https://www.vagabondgame.com): palette swapping.

Palette swapping is simply changing the palette of a texture. Here, we will do that at runtime using shaders. It was a useful technique in the old days to add variety in the assets without using too much memory. Now, it is used in procedural generation to produce new assets, I will show many examples in later devlogs.

![](/media/img/palette-swapping-with-shaders/body_palette_swapping.gif){: .center-image .modal-image }

<!--more-->

# Preparing the Images

The first step is to prepare your images for palette swapping. In a [raster image](https://en.wikipedia.org/wiki/Raster_graphics), each pixel contains a color. What we would like instead is that each pixel contains the index of its color in a palette. This way, we decouple the structure of the image (the areas with the same color) with the real colors.

In fact, several image formats support this way of storing images. For instance, the [PNG image format](https://en.wikipedia.org/wiki/Portable_Network_Graphics#Pixel_format) has an indexed color option. Unfortunately, many libraries that load images will provide an array of colors even if the image was stored in indexed mode. It is the case of SFML, the library I used. It uses [stb_image](https://github.com/nothings/stb) under the hood which automatically "depalettizes" images i.e. it replaces indices by the corresponding color in the palette.

Consequently, to avoid this problem, I store separately the image and the palette. The image is in grayscale mode and the gray level of each pixel corresponds to the index of its color in the palette.

Here is an example of what we expect:

![](/media/img/palette-swapping-with-shaders/preprocess.png){: .center-image .modal-image }

To do that, I use a little Python function that uses the [Pillow](https://github.com/python-pillow/Pillow) library:

```python
import io
import numpy as np
from PIL import Image

def convert_to_indexed_image(image, palette_size):
    # Convert to an indexed image
    indexed_image = image.convert('RGBA').convert(mode='P', dither='NONE', colors=palette_size) # Be careful it can remove colors
    # Save and load the image to update the info (transparency field in particular)
    f = io.BytesIO()
    indexed_image.save(f, 'png')
    indexed_image = Image.open(f)
    # Reinterpret the indexed image as a grayscale image
    grayscale_image = Image.fromarray(np.asarray(indexed_image), 'L')
    # Create the palette
    palette = indexed_image.getpalette()
    transparency = list(indexed_image.info['transparency'])
    palette_colors = np.asarray([[palette[3*i:3*i+3] + [transparency[i]] \
        for i in range(palette_size)]]).astype('uint8')
    palette_image = Image.fromarray(palette_colors, mode='RGBA')
    return grayscale_image, palette_image
```

Firstly, the function converts the image to the palette mode. Then, it reinterprets it as a grayscale image. Finally, it extracts the palette. Nothing fancy, all the hard work is done by Pillow.

# Shader

Now, that we have preprocessed our images, we are ready to write the shader to finally swap the palettes. There are two strategies to pass the palette to the shader: by using a texture or a uniform array. I find that it is easier to do it using a uniform array so I use that.

Here is my shader, I use GLSL but I think you can easily translate it in another shading language as it is dead simple:

```glsl
#version 330 core
in vec2 TexCoords;

uniform sampler2D Texture;
uniform vec4 Palette[32];

out vec4 Color;

void main()
{
    Color = Palette[int(texture(Texture, TexCoords).r * 255)];
}
```

We just use the texture to read the red channel of the current texel. The red channel value is a floating-point number between 0 and 1 so we multiply by 255 and we cast to `int` to retrieve the original gray level between 0 and 255 that is stored in the image. Finally, we used that to get the color from the palette.

The animation at the beginning of the article comes from in-game screenshots where I use the following palettes to color the body of the character:

![](/media/img/palette-swapping-with-shaders/body_palettes.png){: .center-image .modal-image }

# Conclusion

That is all for palette swapping. I hope it gives you some ideas.

In a later post, I will show you how to procedurally generate palettes to push limits of palette swapping.

See you next week for more!