# Motive <-> Java API

This project is part of a larger project developed at the University of Mary Washington.

The `motive` package defines three classes that allow for receiving
rigid body data from OptiTrack Motive version 2.1.1.

The code in this project was adapted from the PythonClient
sample code found in the Motive NatNedSDK. Some minor
reverse engineering was required to determine how to
adapt the code to version 2.1.1.

Note that this code could be adapted to other versions of Motive by
viewing the PythonClient code. Additionally, support for other
frame data, such as skeletons, can be implemented by modifying the code.
The scope of the UMW project only required rigid body data.

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
for each rigid body that exists within the frame, per frame. The id of each rigid body, its x, y, and z coordinates, and its rotational information in the form of a
quaternion, are sent in each `rigidBodyUpdateReceived` call.

After rigid body information is finished processing, a `CommandStreamManager` then
updates all of its `FrameUpdateListener`s via their `frameUpdateReceived`
method.

To use these listener interfaces, simply create a class that implements the two
listener interfaces, like so:

    import motive.*;

    public class ExampleListener implements RigidBodyUpdateListener, FrameUpdateListener {

        @Override
        public void rigidBodyUpdateReceived(int id, float x, float y, float z, 
                float qw, float qx, float qy, float qz) {
            System.out.printf("Rigid body %d moved to %.2f, %.2f, %.2f\n",
                    id, x, y, z);
        }

        @Override
        public void frameUpdateReceived() {
            System.out.println("Frame finished processing");
        }
    }

## Creating a `CommandStreamManager`

After implementing the two listeners above, the next step is to create
a new `CommandStreamManager` object, add your listener to its listener
lists, and wrap the manager object in a new `Thread` object.

We can adapt the code above to do just that:

    import motive.*;

    public class ExampleListener implements RigidBodyUpdateListener, FrameUpdateListener {

        public ExampleListener() {
            CommandStreamManager m = new CommandStreamManager();
            m.addRigidBodyUpdateListener(this);
            m.addFrameUpdateListener(this);
            new Thread(m).start(); // kick things off
        }

        @Override
        public void rigidBodyUpdateReceived(int id, float x, float y, float z, 
                float qw, float qx, float qy, float qz) {
            System.out.printf("Rigid body %d moved to %.2f, %.2f, %.2f\n",
                    id, x, y, z);
        }

        @Override
        public void frameUpdateReceived() {
            System.out.println("Frame finished processing");
        }
    }

This line:

    new Thread(m).start(); // kick things off

is where the magic happens. This causes the newly-created `CommandStreamManager`'s
`run` method to be called from a new thread. A connection to Motive is established
and your two update received methods will start getting called as Motive sends
frame data to your program.

## Program design

One more thing to consider is how to design your program around these listener
methods.

It's recommended that both `FrameUpdateListener` and `RigidBodyUpdateListener`
methods are implemented unless only a single rigid body is present in your scene.

It's also recommended that the `rigidBodyUpdateReceived` method is used to track
the movement of rigid bodies in the scene, while the `frameUpdateReceived` method
is used to update the program once rigid body movement has been accounted for.

An example program can be found below:
https://github.com/lk-umw-cpsc/parking-sim

In the above program, `rigidBodyUpdateReceived` is used to update the location
of objects in the scene, while `frameUpdateReceived` is used to redraw the user
interface once per frame. This design is nice because the order of events causes
the interface to be redrawn only after each rigid body's location (and rotation)
are updated.