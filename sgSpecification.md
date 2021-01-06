# sg Specification

I created this specification by reverse engineering the sgreader found here:
https://github.com/bvschaik/citybuilding-tools

The format uses little endian.

At a high level the assets system is made up of sg files (c3.sg2 for example)
and 555 files. 

The 555 files contain the actual image pixel data and the sg files contain 
meta data about the images and in which 555 file you can find the image data.

The sg files themselfs have two major parts. One is a list of Image Metadata
needed to render an image. The other is what I will call an Image Group 
(aka Bitmap in other tools). Image Groups groups images together and specify 
their source 555 file and the number of images in the group. It should be 
noted that the sg files contain no actual pixel data for the image.

The 555 files are just raw pixel data.

## sg Files

* Image Group List Size
    * The size in bytes of the maximum number of Image Groups that can be stored
      by this file format. See the Image Group List section for how to get the
      Max Image Groups number.      
    * This value is X * (Max Image Groups)

| Name                | Type                | Size (bytes)     | Offset (bytes)                 |
|---------------------|---------------------|------------------|--------------------------------|
| Header              | Header              | 680              | 0x0000                         |
| Image Group List    | Image Group List    | Variable         | 0x02A8                         |
| Image Metadata List | Image Metadata List | Variable         | 0x02A8 + Image Group List Size |

### Header

The Header starts at the begining of the file so the Offset is in refrence to 
the start of the file. 

| Name                                         | Type         | Size (bytes) | Offset (bytes) |
|----------------------------------------------|--------------|--------------|----------------|
| File Size                                    | unsigned int | 4            | 0x0000         |
| Version                                      | unsigned int | 4            | 0x0004         |
| Max Image Records                            | int          | 4            | 0x000C         |
| Number of Image Metadata Records             | int          | 4            | 0x0010         |
| Number of Image Group Records                | int          | 4            | 0x0014         |
| Number of Image Group Records Without System | int          | 4            | 0x0018         |
| Total Filesize                               | unsigned int | 4            | 0x001C         |
| Filesize 555                                 | unsigned int | 4            | 0x0020         |
| Filesize External                            | unsigned int | 4            | 0x0024         |

The decription of the fields are as follows:

* File Size
    * In sg2 files, this is either 74480 or 522680 depending on wheater it's a 
      "normal" sg2 or an "enemy" sg2. [TODO: Find out which value is for what]
    * In sg3 files this is the actual file size in bytes
* Version
    * 0xCF or 0xD3 indicates the file is an sg2 file. 
      [TODO: Find out what the diffrent values mean. Maybe one for normal one
      for enemy?]
    * 0xD5 or 0xD6 indicates the file is an sg3 file.
    * 0xCF  seems to indicate some kind of demo sg2 file format?
    * 0xD6 or greater notes that alpha data is included in the 555 files.
* Max Image Records
    * [TODO: more investigation needed]
* Number of Image Metadata Records
    * Total Number of Images
* Number of Image Group Records 
    * Number of Image Group Records in this file
* Number of Image Group Records Without System
    * [TODO: more investigation needed]
* Total Filesize
    * [TODO: more investigation needed]
* Filesize 555
    * [TODO: more investigation needed]
* Filesize External
    * [TODO: more investigation needed]

### Image Group List

The Image Group List is a list of Image Groups tightly packed. 
The Image Group List has a maximum number of Image Groups set by using the 
following rules:

| Header Version Number | Max Image Groups |
|-----------------------|------------------|
| 0xCF                  | 50               |
| 0xD3                  | 100              |
| 0xD5                  | 200              |
| 0xD6                  | 200              |

Note that the Image Group List may be mostly empty if there isn't many 
Image Groups

### Image Group

Each Image Group has a size of 0xC8 so you can see there is some extra stuff
after the table below for the Image Group. Each Image Group is packed one after 
the other. 

| Name                 | Type         | Size (bytes) | Offset (bytes) |
|----------------------|--------------|--------------|----------------|
| Filename             | string       | 65           | 0x0000         |
| Comment              | string       | 51           | 0x0041         |
| Width                | unsigned int | 4            | 0x0074         |
| Height               | unsigned int | 4            | 0x0078         |
| Number Of Images     | unsigned int | 4            | 0x007C         |
| Start Index          | unsigned int | 4            | 0x0080         |
| End Index            | unsigned int | 4            | 0x0084         |

The decription of the fields are as follows:

* Filename
    * The path to the 555 file that contains the pixel data for this 
      Image Group
    * Appears to be a null terminated string, but I would force terminate 
      at the end of the buffer just in case.
* Comment
    * My guess is just a string location for random information.
    * Appears to be a null terminated string, but I would force terminate 
      at the end of the buffer just in case.
* Width
    * [TODO: more investigation needed]
* Height 
    * [TODO: more investigation needed]
* Number Of Images
    * [TODO: more investigation needed]
* Start Index 
    * [TODO: more investigation needed]
* End Index
    * [TODO: more investigation needed]


The Image Group has an implied Index based on its location in the 
Image Group List.

The following are comments left in sgbitmap.cpp (The source of this document)
ater reading out all the data:

```
	/* 4 bytes - quint32 between start & end */
	/* 16b, 4x int with unknown purpose */
	/*  8b, 2x int with (real?) width & height */
	/* 12b, 3x int: if any is non-zero: internal image */
	/* 24 more misc bytes, most zero */
```

### Image Metadata List

The Image Metadata List is a list of Image Metadata all tightly packed. 
The number of Image Metadata records can be found by the value 
Number of Image Metadata Records in the Header section.

### Image Metadata

Item Metadata can either contain or not contain alpha data. This
depends on the version number from the Header section. If the version number
is greater than or equal to 0xD6, then the Item Metadata contain alpha data.

Image Metadata has a size of 0x40 if it does **NOT** contain alpha data.
Image Metadata has a size of 0x48 if it does contain alpha data.

Each Image Metadata is packed one after the other. 

Without alpha data the structure looks like this:

| Name                    | Type         | Size (bytes) | Offset (bytes) |
|-------------------------|--------------|--------------|----------------|
| Offset                  | unsigned int | 4            | 0x0000         |
| Length                  | unsigned int | 4            | 0x0004         |
| Uncompressed Length     | unsigned int | 4            | 0x0008         |
| Horizontally Mirrored   | boolean      | 4            | 0x0010         |
| Width                   | int          | 2            | 0x0014         |
| Height                  | int          | 2            | 0x0016         |
| Type                    | unsigned int | 2            | 0x0032         |
| External Flag           | boolean      | 1            | 0x0034         |
| Isometric Size Flag     | int          | 1            | 0x0037         |
| Image Group Id          | unsigned int | 1            | 0x0038         |


With alpha data the structure is the same as above, but with the following 
added to the bottom:

| Name                    | Type         | Size (bytes) | Offset (bytes) |
|-------------------------|--------------|--------------|----------------|
| Alpha Offset            | unsigned int | 4            | 0x0040         |
| Alpha Length            | unsigned int | 4            | 0x0044         |

The decription of the fields are as follows:

* Offset
    * The byte offset into the 555 file where the pixel data starts
      for this image.
* Length
    * The size in bytes of the image pixel data in the 555 file.
* Uncompressed Length
    * [TODO: more investigation needed]
* Horizontally Mirrored
    * If true, the image is stored as Horizontally Mirrored and needs to be
      mirrored on the X axis before displayed.
* Width
    * The width of the image in pixels
* Height
    * The height of the image in pixels
* Type
    * Specifies how the pixel data is packed in the 555 file.
    * 0, 1, 10, 12, 13 = Plain Image
    * 30 = Isometric Image
    * 256, 257, 276 = Sprite Image
* External Flag
    * Deals with the 555 file name.
    * If true, then the 555 filename will be the Image Group Filename
    * If false, then the 555 filename will be the sg filename with the 
      file extension changed to 555
* Isometric Size Flag 
    * [TODO: more investigation needed]
* Image Group Id
    * The index of the Image Group that owns this Image.
* Alpha Offset
    * The byte offset into the 555 file where the alpha pixel starts.
* Length
    * The size in bytes of the alpha pixel data in the 555 file.

## 555 Files

These files are binary dumps for image data. The image data means diffrent
things based on the Image Metadata.

### Pixel Data

Color Pixel data is stored in 15 bits io 2 bytes per pixel. Alpha
is not included in Color Pixels. As far as I know the 16th bit has no 
meaning.

| Color Channel | Size (bits) | Offset (bits) |
|---------------|-------------|---------------|
| Red           | 5           | 0             |
| Green         | 5           | 5             |
| Blue          | 5           | 10            |

Alpha Pixel data is stored in 5 bits with 1 byte per pixel. As far as I know 
bits 6-8 have no meaning.

| Color Channel | Size (bits) | Offset (bits) |
|---------------|-------------|---------------|
| Alpha         | 5           | 0             |

### Pixel Data Compression Omega

Pixel data may be compressed. I have made up the name Omega to identify this 
kind of pixel data compression. This compression may be used to store 
Color Pixel or Alpha Pixel data.

Given some bytes representing either Color Pixel or Alpha Pixel data (not both)
you can decompress with the following algorithm:

```
   imageWidth = GetImageWidth();
   pixel_x = 0;
   pixel_y = 0;
   while(HasMoreBytes())
   {
      cmd = ReadByteFromData();
      if(cmd = 255)
      {
         // The command is skip, so the next byte is the number of 
         // pixels to skip
         pixelSkip = ReadByteFromData();
         pixel_x += pixelSkip;
         while(pixel_x >= imageWidth)
         {
            pixel_y ++;
            pixel_x -= imageWidth;
         }
      }
      else
      {
         // The command contains the number of elements to write.
         // Notice that these could be two (color) or one (alpha) byte elements
         while(cmd > 0)
         {
            element = ReadElementFromData();
            DrawPixel(pixel_x, pixel_y, element);
            pixel_x ++;
            if(pixel_x >= imageWidth)
            {
               pixel_y ++;
               pixel_x -= imageWidth;
            }
            cmd --;
         }
      }
   }
```

