# BitVisualizer

## Props

| Prop name         | Description                                                                                                                                                                                                                                                                                                                                                                                           | Type       | Values | Default     |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------ | ----------- |
| data              |                                                                                                                                                                                                                                                                                                                                                                                                       | Uint8Array | -      |             |
| cols              | The number of bits to be rendered in each row (the number of rows is determined by the data length divided by<br/>the number of columns)                                                                                                                                                                                                                                                              | number     | -      | 8           |
| cellSize          | The pixel size of each bit (values of less than one are accepted)                                                                                                                                                                                                                                                                                                                                     | number     | -      | 1           |
| barSimilarity     | If barSimilarity &gt;0 then we display pixels with colors that can represent<br/>a bar chart indicating the number of matching bits                                                                                                                                                                                                                                                                   | number     | -      | 0           |
| pixelColor        | The normal color of bits set to 1 in the binary embedding                                                                                                                                                                                                                                                                                                                                             | string     | -      | '#3937f18c' |
| noMatchPixelColor | The color of bits set to 1 in the binary embedding where 'barSimilarity' is greater than zero<br/>and the bit is non-matching.<br/>Rather than rendering a true representation of which bits did not match in a search (this would<br/>tend to blend colors) the bits are shown as a form of horizontal bar chart with all<br/>1 bits to the right of the bar end displayed in this "no match" color. | string     | -      | 'lightgray' |
| backgroundColor   | The color of bits set to 0                                                                                                                                                                                                                                                                                                                                                                            | string     | -      | 'white'     |

## Expose

### redraw

>

---
