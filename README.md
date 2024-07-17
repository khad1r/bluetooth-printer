# Bluetooth Printer Web App

## Overview

This repository contains a web application that allows users to print text and images to a Bluetooth printer using Web Bluetooth. The application is built using HTML and JavaScript.

this is a rough implementation base on [Web Bluetooth Printer - example](https://github.com/WebBluetoothCG/demos/tree/gh-pages/bluetooth-printer)

There are 2 Classes :

1. `ThermalPrinter` Handle connection and print to bluetooth printer.
1. `PrintBitmapData` Convert image to Bitmap required to print image to bluetooth printer. can convert HTML Element to bitmap, this require [dom-to-image](https://github.com/tsayen/dom-to-image) library.

## Features

- Connect to a Bluetooth thermal printer
- Print text and images
- Print preview for image printing
- Customizable dither threshold maps for image processing

## Example Implementation

### Setup

```javascript
  <script src="https://cdnjs.cloudflare.com/ajax/libs/dom-to-image/2.6.0/dom-to-image.min.js"
    integrity="sha512-01CJ9/g7e8cUmY0DFTMcUw/ikS799FHiOA0eyHsUWfOetgbx/t6oV4otQ5zXKQyIrQGTHSmRVPIgrgLcZi/WMA=="
    crossorigin="anonymous" referrerpolicy="no-referrer"></script>
  <script src="https://cdn.jsdelivr.net/gh/khad1r/bluetooth-printer@main/bluetooth-printer.js" defer></script>
```

### ThermalPrinter Class

#### Connecting to a Bluetooth Printer

To connect to a Bluetooth printer, use the `ThermalPrinter` class from `bluetooth-printer.js`:

```javascript
const BTPrinter = new ThermalPrinter();
async function connectPrinter() {
  try {
    await BTPrinter.connect();
    console.log("Connected to printer");
  } catch (error) {
    console.error("Error connecting to printer:", error);
  }
}
connectPrinter();
```

you might want to overwrite the `onConnected` and `onDisconnected` function. the `device` params is the bluetooth gatt device ands is passed from inside library.

```javascript
const BTPrinter = new ThermalPrinter();
BTPrinter.onConnected = async (device) => {
  console.log(`connected to ${device.name}`);
  await BTPrinter.sendImageData(await printBitmapData.getImagePrintData());
  await BTPrinter.sendTextData("Some text");
};
BTPrinter.onDiConnected = (device) => {
  handleError(`Disconected from ${device.name}`);
};
BTPrinter.connect();
```

you can pass your own device gatt, it might be from passed item from `onConnected` and `onDisconnected` on previous example.

```javascript
BTPrinter.connect(device);
```

#### Printing Text and Images

To print text and images, use the `sendImageData` and `sendTextData` methods of the `ThermalPrinter` class:

```javascript
const printButton = document.querySelector("#print");
const message = document.querySelector("#message");
const receipt = document.querySelector("#receipt");
const printBitmapData = new PrintBitmapData(receipt);

printButton.addEventListener("click", async () => {
  try {
    await BTPrinter.connectToPrinter();
    await BTPrinter.sendImageData(await printBitmapData.getImagePrintData());
    await BTPrinter.sendTextData(message.value);
    console.log("Print successful");
  } catch (error) {
    console.error("Error printing:", error);
  }
});
```

### PrintBitmapData Class

this class require [dom-to-image](https://github.com/tsayen/dom-to-image) library for converting HTML Dom to Image.

#### HTML Element

```javascript
let receipt = document.querySelector("#receipt");
const printBitmapData = new PrintBitmapData(receipt);
```

you can just pass `<img>` element because it straight use the image itself instead wrap with [dom-to-image](https://github.com/tsayen/dom-to-image)

```javascript
let image = document.querySelector("img#logo");
const printBitmapData = new PrintBitmapData(image);
```

#### pass to `ThermalPrinter`

```javascript
async ()=>{
  printData = await printBitmapData.getImagePrintData();
  BTPrinter.sendImageData(printData).then(()=>{
    console.log('done writing...')
  });
}()
```

#### Customize

for default the paper size i use is 58mm, you can specify on object construction. as for the `thresholdMapIndex` you can check on the file

```javascript
let receipt = document.querySelector("#receipt");
const printBitmapData = new PrintBitmapData(receipt, 3, 80);
```

#### Print Preview

you can use the HTML Element properties from this class

```javascript
const printBitmapData = new PrintBitmapData(receipt);
document.body.appendChild(printBitmapData.imgHTMLElement);
document.body.appendChild(printBitmapData.canvasHTMLElement);
```

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](./LICENSE) file for details.

### Acknowledgements

- Google for the initial template and Web Bluetooth examples.
- [dom-to-image](https://github.com/tsayen/dom-to-image) for converting DOM elements to images.
- OpenAI's ChatGPT for assisting in generating the initial version of the image processing code.

### Contact

For any inquiries or issues, please open an issue on GitHub or contact [me@khadr.dev](me@khadr.dev).
