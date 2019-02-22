# Sharing Names with the micro:bit

## The plan

This simple program will lead to a lot of interesting discussions in your
classroom about networks and their security. Via the radio, a micro:bit
can broadcast a simple message (either a number or a string) to all the other
micro:bits. It is a broadcast as the micro:bit does not get to choose
who can listen to the message. Every micro:bit can broadcast messages 
as well as listen for messages sent by other micro:bits. 

Now, just as TVs have channels, the micro:bit radio has a notion of 
`groups`. That is, a micro:bit can tune in to the messages of a particular 
group (channel), so it won't receive messages sent to other groups. However,
groups are not private. Any micro:bit can listen to the messages for any group.

In this classroom exercise, you'll have your students pair up on computers
and create a simple program to send names to each other, which will
then display their the names on the LED display. When they run this
program in the simulator, it will allow Alice to send her name to Bob
and Bob to send his name to Alice.  But we will instruct students to 
set their micro:bit's radio group to 0.
This means that when your students deploy their program to actual micro:bits,
they will see the names of other students in the classroom besides the student
they are paired with!

## the program

Instruct your students to start with the following program:
``` blocks
radio.setGroup(0)
```
As discussed above, this will ensure that all micro:bits are listening
to the same ''channel'' (if you don't include this block, the
micro:bit compiler will select a random channel, which means that
the micro:bits flashed with the same hex file will listen to the
same group, whereas micro:bits flashed with a different hex file
likely will listen to a different group). 

Here's the program we are aiming each pair of students to create
together (with their names instead of Alice and Bob, of course):

<!-- https://makecode.microbit.org/_V4YRA352V56c -->

``` blocks
radio.setGroup(0)

input.onButtonPressed(Button.A, function () {
    radio.sendString("Alice")
})

input.onButtonPressed(Button.B, function () {
    radio.sendString("Bob")
})

radio.onReceivedString(function (receivedString) {
    basic.showString(receivedString)
})
```

## disjoint groups

## channel hopping

## encryption
