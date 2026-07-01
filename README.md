# Add-Legends-and-Layer-Labels-to-a-Circos-Plot-Python-Pillow-
This repository contains a Python script for adding a publication-ready legend and layer labels to an existing Circos plot. The script assumes that the Circos figure has already been generated and combines it with a separate legend image, places the legend beneath the plot, adds customizable layer labels, and exports the final figure in PNG, TIFF (600 dpi), and PDF formats.

## Requirements
```bash
- Python ≥ 3.13
- Pillow (PIL)

On the HPC system:
ml python/3.13.8

Install Pillow if needed:
pip install pillow
```

## Input
```bash
Assuming you already have a completed Circos plot and a legend image:
- Circos plot (PNG)
- Legend image (TIFF or PNG)
Example:
2circos-wheat.png
circos_legend.tiff
```

## Features
```bash
- Combines an existing Circos plot with a legend
- Automatically resizes the legend
- Places the legend below the Circos plot
- Adds customizable layer labels (a–g)
- Allows manual adjustment of label positions
- Automatically detects a bold system font
- Exports publication-quality figures as:
    - PNG
    - TIFF (600 dpi with LZW compression)
    - PDF
```

## Run python
```bash
ml python/3.13.8

nano 3x_circos_legend_combined.py

from PIL import Image, ImageDraw, ImageFont
import os


# -----------------------------
# Input files
# -----------------------------
circos_path = "/90daydata/oat_renewal/Ruby/circos_plot_wheat/2circos-wheat.png"
legend_path = "/90daydata/oat_renewal/Ruby/circos_plot_wheat/circos_legend.tiff"


# Output files
output_png = "/90daydata/oat_renewal/Ruby/circos_plot_wheat/2circos-wheat_with_legend_labels.png"
output_tiff = "/90daydata/oat_renewal/Ruby/circos_plot_wheat/2circos-wheat_with_legend_labels_600dpi.tiff"
output_pdf = "/90daydata/oat_renewal/Ruby/circos_plot_wheat/2circos-wheat_with_legend_labels.pdf"


# -----------------------------
# Settings
# -----------------------------
legend_scale_factor = 0.28
legend_right_margin = 50
extra_bottom_space = 150
legend_gap_below_circle = 0


labels = ["a", "b", "c", "d", "e", "f", "g"]


label_x_fractions = [
    0.503,
    0.504,
    0.505,
    0.506,
    0.505,
    0.504,
    0.503
]


label_y_fractions = [
    0.080,
    0.112,
    0.142,
    0.172,
    0.202,
    0.232,
    0.262
]


label_size = 30


# -----------------------------
# Load images
# -----------------------------
circos_img = Image.open(circos_path).convert("RGB")
legend_img = Image.open(legend_path).convert("RGBA")


W, H = circos_img.size
print("Original image size:", W, H)


# -----------------------------
# Resize legend
# -----------------------------
new_legend_width = int(W * legend_scale_factor)
new_legend_height = int(new_legend_width * legend_img.height / legend_img.width)


legend_img = legend_img.resize(
    (new_legend_width, new_legend_height),
    Image.Resampling.LANCZOS
)


# -----------------------------
# Create larger canvas
# -----------------------------
final_W = W
final_H = H + extra_bottom_space


final_img = Image.new("RGB", (final_W, final_H), "white")
final_img.paste(circos_img, (0, 0))


# -----------------------------
# Add legend below circle
# -----------------------------
legend_x = final_W - new_legend_width - legend_right_margin
legend_y = H + legend_gap_below_circle


final_img.paste(legend_img, (legend_x, legend_y), legend_img)


# -----------------------------
# Add layer labels
# -----------------------------
draw = ImageDraw.Draw(final_img)


font_candidates = [
    "/usr/share/fonts/dejavu/DejaVuSans-Bold.ttf",
    "/usr/share/fonts/liberation/LiberationSans-Bold.ttf",
    "/usr/share/fonts/liberation2/LiberationSans-Bold.ttf",
    "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf"
]


font_path = None
for f in font_candidates:
    if os.path.exists(f):
        font_path = f
        break


if font_path is not None:
    font = ImageFont.truetype(font_path, label_size)
    print("Using font:", font_path)
else:
    try:
        font = ImageFont.truetype("DejaVuSans-Bold.ttf", label_size)
        print("Using Pillow DejaVuSans-Bold.ttf")
    except:
        font = ImageFont.load_default()
        print("WARNING: using default font, labels may be small")


print("Label size:", label_size)


label_x = int(W * label_x_fraction)


for lab, y_fraction in zip(labels, label_y_fractions):
    label_y = int(H * y_fraction)


    bbox = draw.textbbox((label_x, label_y), lab, font=font, anchor="mm")
    padding = int(label_size * 0.15)


    draw.rectangle(
        [
            bbox[0] - padding,
            bbox[1] - padding,
            bbox[2] + padding,
            bbox[3] + padding
        ],
        fill="white"
    )


    draw.text(
        (label_x, label_y),
        lab,
        fill="black",
        font=font,
        anchor="mm"
    )


# -----------------------------
# Save outputs
# -----------------------------
final_img.save(output_png, format="PNG")


final_img.save(
    output_tiff,
    format="TIFF",
    dpi=(600, 600),
    compression="tiff_lzw"
)


final_img.save(
    output_pdf,
    "PDF",
    resolution=600.0
)


print("Saved:")
print(output_png)
print(output_tiff)
print(output_pdf)

python 3x_circos_legend_combined.py
```

## Output
```bash
The script generates three publication-ready files:
- 2circos-wheat_with_legend_labels.png
- 2circos-wheat_with_legend_labels_600dpi.tiff
- 2circos-wheat_with_legend_labels.pdf

The TIFF image is exported at 600 dpi using LZW compression, making it suitable for journal submission while reducing file size.
```

Maintainer:

Rubylyn Mijan
