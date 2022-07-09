# Motive <-> Java API

This project is part of a larger project developed at the University of Mary Washington.

The `motive` package defines three classes that allow for receiving
rigid body data from OptiTrack Motive version 2.1.1.

The code in this project was adapted from the PythonClient
sample code found in the Motive NatNedSDK. Some minor
reverse engineering was required to determine how to
adapt the code to version 2.1.1.

# Using the API

## Implement appropriate listeners

Two listener classes are used by the API to update your
program when rigid body data is received by Motive
and/or a frame has finished being processed by the
API's network processing thread.

`RigidBodyUpdateListener` is an interface that can be implemented
to listen for rigid body updates, while `FrameUpdateListener`
can be implemented to know when a frame was received and processed.

A `CommandStreamManager` object will notify all of its `RigidBodyUpdateListener`s
via their `rigidBodyUpdateReceived` method. This method is called once
for each rigid body that exists within the frame. The x, y, and z coordinates
of each rigid body, along with their rotational information in the form of a
quaternion, are sent in each `rigidBodyUpdateReceived` call.

After rigid body information is finished processing, a `CommanStreamManager` then
updates all of its `FrameUpdateListener`s via their `frameUpdateReceived`
method.

To use these listener interfaces, simply create a class that implements the two
listener interfaces, like so:

`public class ExampleListener implements RigidBodyUpdateListener, FrameUpdateListener {`
` `
`   public void rigidBodyUpdateReceived(int id, float x, float y, float z, float qw, float qx, float qy, float qz) {`
`   // your code here`
    `}`
` `
`   public void frameUpdateReceived() {`
`       // your code here`
`   }`
`}`