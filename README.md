TITLE: HIDING A TEXT INSIDE AN IMAGE USING Steganogphy 

Skip to content
Navigation Menu

Code
Issues
Pull requests
 0 stars
 0 forks
 1 watching
 1 Branch
 0 Tags
 Activity
Public template repository
rachana-palo/STEGANOGRAPHY
Folders and files
Name	
Latest commit
rachana-palo
rachana-palo
1 hour ago
History
README.md
2 hours ago
image1.jpg
2 hours ago
image2.jpg
2 hours ago
requirement.txt
2 hours ago
steganography.py
2 hours ago
Repository files navigation
README
Steganography: Hiding a text inside an image
Usage
Create a virtualenv and install the requirements:

virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
Then, merge and unmerge your files with:

python steganography.py merge --image1=res/image1.jpg --image2=res/image2.jpg --output=res/output.png
python steganography.py unmerge --image=res/output.png --output=res/output2.png
To use the Steganography class in your Python code, you will need to use the Image module from the Pillow library, for example:

from PIL import Image

merged_image = Steganography().merge(Image.open(image1), Image.open(image2))
merged_image.save(output)
Note: the output image from the merge operation and the input image for the unmerge operation must be in PNG format.

Steganography
Let’s understand what is steganography, digital images, pixels, and color models.

What is steganography?
Steganography is the practice of concealing a file, message, image, or video within another file, message, image, or video.

What is the advantage of steganography over cryptography?
The advantage of steganography over cryptography alone is that the intended secret message does not attract attention to itself as an object of scrutiny. Plainly visible encrypted messages, no matter how unbreakable they are, arouse interest and may in themselves be incriminating in countries in which encryption is illegal. In other words, steganography is more discreet than cryptography when we want to send a secret information. On the other hand, the hidden message is easier to extract.

What is a digital image?
Ok, now that we know the basics of steganography, let’s learn some simple image processing concepts.

Before understanding how can we hide an image inside another, we need to understand what a digital image is.

We can describe a digital image as a finite set of digital values, called pixels. Pixels are the smallest individual element of an image, holding values that represent the brightness of a given color at any specific point. So we can think of an image as a matrix (or a two-dimensional array) of pixels which contains a fixed number of rows and columns.



When using the “digital image” term here, we are referencing to the “raster graphics”, which are basically a dot matrix data structure, representing a grid of pixels, which in turn can be stored in image files with varying formats. You can read more about digital images, raster graphics, and bitmaps at the Wikipedia website.

Pixel concept and color models
As already mentioned, pixels are the smallest individual element of an image. So, each pixel is a sample of an original image. It means, more samples provide more accurate representations of the original. The intensity of each pixel is variable. In color imaging systems, a color is typically represented by three or four component intensities such as red, green, and blue, or cyan, magenta, yellow, and black.

Here, we will work with the RGB color model. As you can imagine, the RGB color model has 3 channels, red, green and blue.

The RGB color model is an additive color model in which red, green and blue light are added together in various ways to reproduce a broad array of colors. The name of the model comes from the initials of the three additive primary colors, red, green, and blue. The main purpose of the RGB color model is for the sensing, representation and display of images in electronic systems, such as televisions and computers, though it has also been used in conventional photography. 

So, each pixel from the image is composed of 3 values (red, green, blue) which are 8-bit values (the range is 0–255).



As we can see in the image above, for each pixel we have three values, which can be represented in binary code (the computer language).

When working with binary codes, we have more significant bits and less significant bits, as you can see in the image below.



The leftmost bit is the most significant bit. If we change the leftmost bit it will have a large impact on the final value. For example, if we change the leftmost bit from 1 to 0 (11111111 to 01111111) it will change the decimal value from 255 to 127.

On the other hand, the rightmost bit is the least significant bit. If we change the rightmost bit it will have less impact on the final value. For example, if we change the leftmost bit from 1 to 0 (11111111 to 11111110) it will change the decimal value from 255 to 254. Note that the rightmost bit will change only 1 in a range of 256 (it represents less than 1%).

Summarizing: each pixel has three values (RGB), each RGB value is 8-bit (it means we can store 8 binary values) and the rightmost bits are least significant. So, if we change the rightmost bits it will have a small visual impact on the final image. This is the steganography key to hide an image inside another. Change the least significant bits from an image and include the most significant bits from the other image.



You can check out the result in the following image:

The left upper image is the image that will hide the right upper image. The left lower image is the two images merged and the right lower image is the extracted (unmerged) image.



As you can see in the image above, we lost some image quality in the process, but this does not interfere with image comprehension. Binary file addedBIN +921 KB image1.jpg Loading Binary file addedBIN +500 KB image2.jpg Loading 2 changes: 2 additions & 0 deletions2
requirements.txt Original file line number Diff line number Diff line change @@ -0,0 +1,2 @@ Pillow==10.3.0 click==8.0.1 123 changes: 123 additions & 0 deletions123
steganography.py Original file line number Diff line number Diff line change @@ -0,0 +1,123 @@ import argparse

from PIL import Image

class Steganography:

BLACK_PIXEL = (0, 0, 0)

def _int_to_bin(self, rgb):
    """Convert an integer tuple to a binary (string) tuple.
    :param rgb: An integer tuple like (220, 110, 96)
    :return: A string tuple like ("00101010", "11101011", "00010110")
    """
    r, g, b = rgb
    return f'{r:08b}', f'{g:08b}', f'{b:08b}'

def _bin_to_int(self, rgb):
    """Convert a binary (string) tuple to an integer tuple.
    :param rgb: A string tuple like ("00101010", "11101011", "00010110")
    :return: Return an int tuple like (220, 110, 96)
    """
    r, g, b = rgb
    return int(r, 2), int(g, 2), int(b, 2)

def _merge_rgb(self, rgb1, rgb2):
    """Merge two RGB tuples.
    :param rgb1: An integer tuple like (220, 110, 96)
    :param rgb2: An integer tuple like (240, 95, 105)
    :return: An integer tuple with the two RGB values merged.
    """
    r1, g1, b1 = self._int_to_bin(rgb1)
    r2, g2, b2 = self._int_to_bin(rgb2)
    rgb = r1[:4] + r2[:4], g1[:4] + g2[:4], b1[:4] + b2[:4]
    return self._bin_to_int(rgb)

def _unmerge_rgb(self, rgb):
    """Unmerge RGB.
    :param rgb: An integer tuple like (220, 110, 96)
    :return: An integer tuple with the two RGB values merged.
    """
    r, g, b = self._int_to_bin(rgb)
    # Extract the last 4 bits (corresponding to the hidden image)
    # Concatenate 4 zero bits because we are working with 8 bit
    new_rgb = r[4:] + '0000', g[4:] + '0000', b[4:] + '0000'
    return self._bin_to_int(new_rgb)

def merge(self, image1, image2):
    """Merge image2 into image1.
    :param image1: First image
    :param image2: Second image
    :return: A new merged image.
    """
    # Check the images dimensions
    if image2.size[0] > image1.size[0] or image2.size[1] > image1.size[1]:
        raise ValueError('Image 2 should be smaller than Image 1!')

    # Get the pixel map of the two images
    map1 = image1.load()
    map2 = image2.load()

    new_image = Image.new(image1.mode, image1.size)
    new_map = new_image.load()

    for i in range(image1.size[0]):
        for j in range(image1.size[1]):
            is_valid = lambda: i < image2.size[0] and j < image2.size[1]
            rgb1 = map1[i ,j]
            rgb2 = map2[i, j] if is_valid() else self.BLACK_PIXEL
            new_map[i, j] = self._merge_rgb(rgb1, rgb2)

    return new_image

def unmerge(self, image):
    """Unmerge an image.
    :param image: The input image.
    :return: The unmerged/extracted image.
    """
    pixel_map = image.load()

    # Create the new image and load the pixel map
    new_image = Image.new(image.mode, image.size)
    new_map = new_image.load()

    for i in range(image.size[0]):
        for j in range(image.size[1]):
            new_map[i, j] = self._unmerge_rgb(pixel_map[i, j])

    return new_image
def main(): parser = argparse.ArgumentParser(description='Steganography') subparser = parser.add_subparsers(dest='command')

merge = subparser.add_parser('merge')
merge.add_argument('--image1', required=True, help='Image1 path')
merge.add_argument('--image2', required=True, help='Image2 path')
merge.add_argument('--output', required=True, help='Output path')

unmerge = subparser.add_parser('unmerge')
unmerge.add_argument('--image', required=True, help='Image path')
unmerge.add_argument('--output', required=True, help='Output path')

args = parser.parse_args()

if args.command == 'merge':
    image1 = Image.open(args.image1)
    image2 = Image.open(args.image2)
    Steganography().merge(image1, image2).save(args.output)
elif args.command == 'unmerge':
    image = Image.open(args.image)
    Steganography().unmerge(image).save(args.output)
if name == 'main': main()

Releases
No releases published
Packages
No packages published
Languages
Python
100.0%
Footer
© 2024 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Docs
Contact
Manage cookies
Do not share my personal information
