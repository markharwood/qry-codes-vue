# ClusterSlider

## Props

| Prop name  | Description                                                                                                                                                                                                                                                                                                                                      | Type   | Values | Default |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------ | ------ | ------- |
| min        | The minimum similarity connecting all vectors in a cluster.<br/>In practice acceptable values range from 0.5 to 1.<br/>A binary vector resembling random white noise (think of an untuned TV) would <br/>match 50% of bits in typical vectors. The mininum value of 0.5 therefore typically<br/>links all provided vectors into a single cluster | number | -      | 0.5     |
| max        | The maximum similarity connecting all vectors in a cluster.<br/>A value of 1 typically represents identical content and is there the max similarity                                                                                                                                                                                              | number | -      | 0.99    |
| modelValue | The property which will hold the value of the slider's current position                                                                                                                                                                                                                                                                          | number | -      |         |
| thumbSize  | The size in the pixels of the thumb track shown in the slider - larger values help on mobile devices                                                                                                                                                                                                                                             | number | -      | 18      |

## Events

| Event name        | Properties | Description |
| ----------------- | ---------- | ----------- |
| mounted           |            |
| update:modelValue |            |

## Slots

| Name       | Description                                                                                                 | Bindings |
| ---------- | ----------------------------------------------------------------------------------------------------------- | -------- |
| header     | Slot used for the description shown above the slider input                                                  |          |
| leftLabel  | Slot used for the description shown to the left of the slider input to indicate low-similarity clustering   |          |
| rightLabel | Slot used for the description shown to the right of the slider input to indicate high-similarity clustering |          |

---
