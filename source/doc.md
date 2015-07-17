
#JSON document descriptors - overview
In some cases you - as an integrator - migth want to use the raw document format instead of using the editor and fetch the document from the created project. Some example cases could be when you want to set custom headers and footers to some of your users or you want to create complex or dynamic element.

We can distiguish two main element descriptors, from which a whole document can be composed. These two main types are the containers and the leaf elements.

Every element has its type property. The possible values are the followings: "BOX", "MULTICOLUMN", "ROOT", "TEXT", "TITLE", "IMAGE", "BUTTON and "FULLWIDTH_CONTAINER".

#Containers
Containers are used to group some elements and define the layout of the project. You can put any kind of elements into containers, so these elements can be other containers and leaf elements as well.
##Box
Boxes are great to hold other elements together, give them margins, paddings, borders and/or background.
Example:

  {
    type: "BOX",
    margin: {
      top: 0,
      right: 20,
      bottom: 0,
      left: 20
    },
    padding: {
      top: 10,
      right: 10,
      bottom: 10,
      left: 10
    },
    border: {
      top: {
        style: "solid",
        width: 1,
        color: "#aabbaa"
      },
      right: {
        style: "dotted",
        width: 2,
        color: "#aabbaa"
      },
      bottom: {
        style: "dashed",
        width: 3,
        color: "#aabbaa"
      },
      left: {
        style: "none",
        width: 0,
        color: "#aabbaa"
      }
    },
    background: {
      color: "#00aaff",
      image: {
        src: "http://startups.hu/images/logos/edmdesigner.png",
        repeat: "no-repeat",
        position: "top center"
      }
    },
    children: [
      //puth the child elements here
    ]
  }
  
The properties what you can see in the example above, are optional.

##Multicolumn
Multicolumns are the main elements to create complex layouts. They can have multiple columns (we support max. 5 in our editor) and every columns can have children. Example:

  {
    type: "MULTICOLUMN",
    cols: [
      [
        //children of the first column
      ],
      [
        //children of the second column
      ]
    ]
  }

##Root
Every document starts with this elem, as this is the top parent elem. It contains a full width container as a child what has a box inside as a child. It has no other property out of the children field. ({type: "ROOT", ... }) In the near future - when we will introduce the full width containers - it will change a little bit.  

Example:

  {
    type: "ROOT",
    children: [
      {
        type: "FULLWIDTH_CONTAINER,
        leftChildren: [
          type: "BOX",
          ... any box properties ...
        ],
        order: "LTR",
        twoCell: false
      }
    ]
  }

##Full width container
The main feature of this element type will be that the background color can be full width, not only 600px as the root element.
It can only be a direct child of the root, so can be placed only in the highest level of the document structure. A fullwidth container can have 1 cell, or one left cell and one right cells divided from the middle. The cells can have their own background color. The elem can be set "left to right" or "right to left". In case of "left to right" setting the left cells will be ordered on the top if there is no enough room for both in the same line. In case of "right to left" the right cell will do so. If the element is set to be 1 celled, the order property determines that which cell is displayed. The content of the not visible cell always will be stored so will never be lost only not displayed.    

Example:

  {
    type: "FULLWIDTH_CONTAINER",
    order: "RTL", (or "LTR")
    twoCell: "true", (or "false")
    leftBackgroundColor: "#aaaaaa",
    rightBackgroundColor: "#FF0000",
    leftChildren: [
      ...any element except root and fullwidth container...
    ],
    rightChildren: [
      ...any element except root and fullwidth container...
    ]
  }

#Leaf elements
Leaf elements cannot have any child elements. These elements are typically the content elements.

##Text
The text element must contain 2 fields: 'text' what can contain html code, but you should not put very complex things in i, and the 'type' with constant 'TEXT'. 
In our editor we enable the simple text formatting options (bold, italic, underlined) and lists (ul, ol) and some other very simple things like h1, h2, h3.
The 'linkColor' and 'linkUnderLine' specifies the links styles for those <a> tags what has no inline style. The defaults values can be found in the example below. 
The 'textStyles' specifies the style for those (non H1, H2, H3) text contents what have no inline specified style. The defaults values can be found in the example below. 
The 'h1Styles', 'h2Styles' and 'h3Styles' specifies the style for those (H1, H2, H3) tags what have no inline specified style. The defaults values can be found in the example below. 
The spacing specifies a padding values around the whole text div. The defaults values can be found in the example below. 

 Example:

  {
    text: "yo <i>this</i> is Sparta",
    type: "TEXT",
    linkColor: "#5555ff",
          linkUnderLine: true,
          textStyles: {
              lineHeight: 14,
              color": "#000000",
              size": 14,
              family": "arial"
          },
           h3Styles: {
              lineHeight: 32,
              color": "#000000",
              size": 32,
              family": "arial"
          },
           h2Styles: {
              lineHeight: 26,
              color": "#000000",
              size": 26,
              family": "arial"
          },
           h1Styles: {
              lineHeight: 20,
              color": "#000000",
              size": 20,
              family": "arial"
          },
    "spacing": {
                    "left": "5",
                    "bottom": "5",
                    "right": "5",
                    "top": "5"
                }
  }

##Title
Title elements are just like texts, but we enable the users to set only h1, h2 and h3 as formatting.

Example:

  {
    text: "yo <i>this</i> is Sparta",
    type: "TITLE",
    linkColor: "#5555ff",
          linkUnderLine: true,
           h3Styles: {
              lineHeight: 32,
              color": "#000000",
              size": 32,
              family": "arial"
          },
           h2Styles: {
              lineHeight: 26,
              color": "#000000",
              size": 26,
              family": "arial"
          },
           h1Styles: {
              lineHeight: 20,
              color": "#000000",
              size": 20,
              family": "arial"
          },
    "spacing": {
                    "left": "5",
                    "bottom": "5",
                    "right": "5",
                    "top": "5"
                }
  }

##Image
Images are relatively complex elements. Just like boxes, they can have paddings, margins, borders, background color (no background image), but they can't have children. They have their own, special properties as well:
  - src: the url of the image
  - altText: the alt text
  - link: you can wrapp in a link to your image with this propery
  - originalWidth: the original width of the image
  - originalHeight: the original height of the image
  - width: the scaled width of the image (how width should it be in your newsletter)
  - height: the scaled height of the image (how heigh should it be in your newsletter) - this is a must have property if you want your newsletters to render nice on outlooks

##Button
Button is relatively complex as well. It can have the following box properties: paddings, margins, borders, backgrouns (color and background image as well). In addition it can have radius. Baically button is a fancy link, so it has the following properties as well:
  - text: the text what sould appear (it can be a simple html snippet as well)
  - href: the link.
  - sizeType: "FIXED" - exact amount of pixels, "FIT_TO_TEXT": the width will be based on the width of the text
  - width: num of pixels if the sizeType is "FIXED"
