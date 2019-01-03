# DFU

* Install [nrfutil](https://github.com/NordicSemiconductor/pc-nrfutil)


* package .hex file

```
nrfutil pkg generate --hw-version 51 --sd-req 0 --application microbit-dfu-test.hex --application-version 0 app.zip
```
* display package data

```
nrfutil pkg display app.zip
```