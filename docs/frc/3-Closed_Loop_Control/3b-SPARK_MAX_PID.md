# SPARK MAX PID

With the SPARK MAX PID system, the process is a bit different, but not too complicated. We first create a `CANPIDController` object from the SPARK MAX motor object, and then we assign it a setpoint to reach. Then, the onboard controller will do the calculations to arrive at the setpoint with the loaded constants. We will go over this in detail next. Click [here](http://www.revrobotics.com/content/sw/max/sw-docs/java/com/revrobotics/CANPIDController.html) to find the documentation on the `CANPIDController`.

First, we need to do the same as the `PIDController` implementation by creating our constants.

```java
public final class Constants {

    public static class ElectricalLayout {
        public final static int CONTROLLER_DRIVER_ID = 0;
        public final static int CONTROLLER_OPERATOR_ID = 1;

        public final static int ELEVATOR_PRIMARY_MOTOR_ID = 4;
    }

    public static class Autonomous {

    }

    public static class Elevator {
        public static double[] ELEVATOR_PID = new double[] {0.1, 0, 0.01};
        public static double ELEVATOR_PID_TOLERANCE = 0.1;
        public static double ELEVATOR_PID_MAX_OUTPUT = 0.7;
    }

    public static double PI = 3.14159265;
    public static double UPDATE_PERIOD = 0.010; // seconds
}
```

After this, we can set up the motors and encoder that we will be working with in our elevator subsystem.

```java
public class Elevator extends SnailSubsystem {

    public enum State {
        MANUAL
    }

    private CANSparkMax primaryMotor;
    private CANEncoder primaryEncoder;

    private State defaultState = State.MANUAL;
    private State state = defaultState;
    private double speed;

    public Elevator() {
        primaryMotor = new CANSparkMax(ELEVATOR_PRIMARY_MOTOR_ID, MotorType.kBrushless);
        primaryMotor.restoreFactoryDefaults();

        primaryEncoder = new CANEncoder(primaryMotor);
        primaryEncoder.setPositionConversionFactor(48.0 * Math.PI * 6);
        primaryEncoder.setVelocityConversionFactor(48.0 * Math.PI * 6 / 60);
    }

    ...
}
```

Next, we can set up the `CANPIDController` object that will handle almost all things PID.

```java
public class Elevator extends SnailSubsystem {

    public enum State {
        MANUAL
    }

    private CANSparkMax primaryMotor;
    private CANEncoder primaryEncoder;
    private CANPIDController elevatorPID;

    private State state = defaultState;
    private double speed;

    public Elevator() {
        primaryMotor = new CANSparkMax(ELEVATOR_PRIMARY_MOTOR_ID, MotorType.kBrushless);
        primaryMotor.restoreFactoryDefaults();

        primaryEncoder = new CANEncoder(primaryMotor);
        primaryEncoder.setPositionConversionFactor(48.0 * Math.PI * 6);
        primaryEncoder.setVelocityConversionFactor(48.0 * Math.PI * 6 / 60);

        elevatorPID = new CANPIDController(primaryMotor);
        elevatorPID.setP(ELEVATOR_PID[0]);
        elevatorPID.setI(ELEVATOR_PID[1]);
        elevatorPID.setD(ELEVATOR_PID[2]);
        elevatorPID.setOutputRange(-ELEVATOR_PID_MAX_OUTPUT, ELEVATOR_PID_MAX_OUTPUT);
    }

    ...
}
```

**Quick Note: if you have two different set of PID gains (constants), you can specify an optional parameter after each constant to add the PID slot that you want to use. This allows you to switch between constants if you need to use that feature. Read the [documentation](http://www.revrobotics.com/content/sw/max/sw-docs/java/com/revrobotics/CANPIDController.html) for more info.**

Next, we need to make our `update()` function with the `PID` state. The main difference here is that to make the SPARK MAX start using PID, we go to the `CANPIDController` and just tell it where we want to go. It handles everything else.

Another difference is that while before, we would store the setpoint with the `PIDController` object, we now have to store it in a separate variable because the `CANPIDController` object does **not** store it as of the time of writing. Finally, we have to do the error checking ourselves to see if we have arrived at the desired position.

```java
public class Elevator extends SnailSubsystem {

    public enum State {
        MANUAL,
        PID
    }

    ...

    private double setpoint;

    ...

    @Override
    public void update() {
        switch(state) {
            case MANUAL:
                primaryMotor.set(speed);
            break;
            case PID:
                // send the desired setpoint to the PID controller and specify we want to use position control
                elevatorPID.setReference(setpoint, ControlType.kPosition);

                // check our error and update the state if we finish
                if(Math.abs(primaryEncoder.getPosition() - setpoint) < ELEVATOR_PID_TOLERANCE) {
                    state = State.MANUAL;
                }
            break;
        }
    }
}
```

Finally, we can just make the function that will enable this PID mode:

```java
public void setPosition(double setpoint) {
    state = State.PID;
    this.setpoint = setpoint;
}
```

And with that, we're done with the SPARK MAX PID in our subsystem! Now, we just have to add the command that will actually trigger the `setPosition()` function and tell our subsystem to enter the `PID` state.
