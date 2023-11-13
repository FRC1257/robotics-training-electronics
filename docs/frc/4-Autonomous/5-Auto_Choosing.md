# Auto Choosing

Now that we've written out all of our robot's autonomous functionality, we need to actually *call* it at some point (through Shuffleboard) and command our robot to execute it. This section will be heavily based off 1257's 2020 autos, once again.

Most of the work here is done in `RobotContainer.java`, with a few scheduling necessities in `Robot.java` and some constants.

## Setup

First, we have to consider our robot's autonomous "states." They include the robot's auto drive type (segmented vs. trajectory), starting position, and end goal--in 2020 we did not have to worry about where to score, as the target zone position was constant. (In 2018, however, the scoring goals were on varying sides of the field.)

The robot states are stored as [enums](https://frc1257.github.io/robotics-training/#/java/2-Control_Flow/7-Enums), and are placed in `Constants.java` in the `Autonomous` static class in the following manner:

```java
...
    public static class Autonomous {
        public static enum AutoType {
            SEGMENTED,
            TRAJECTORY
        }

        public static enum AutoPosition {
            TOP,
            MIDDLE,
            BOTTOM
        }

        public static enum AutoGoal {
            DEFAULT,
            TRENCH,
            GEN_BOTTOM,
            GEN_TOP
        }

        public static double SEC_PER_M = 1.0 * 100; // seconds / m for auto
        public static double TURN_ANGLE_TIME = 1.5 * 100;
        public static double INDEXER_DUMP_TIME = 2.0; // seconds
    }
...
```

## RobotContainer

Now that we've created the states, we'll move into `RobotContainer.java`. First, we must use this import statement: `import static frc.robot.Constants.Autonomous.*;` to actually get `Autonomous`'s values.

To actually call the auto routines through Shuffleboard, we must use what is called a `SendableChooser`. This essentially allows us display on Shuffleboard a list of options that a programmer/driver can interact with. Because we want to display the `AutoType`, `AutoPosition`, and `AutoGoal` enums, we'll declare the following:

```java
SendableChooser<AutoType> autoTypeChooser;
SendableChooser<AutoPosition> autoPositionChooser;
SendableChooser<AutoGoal> autoGoalChooser;
```

where the type of the displayed options is specified between the <>.

Now, let's jump to the method `configureAutoChoosers()`. Just as it says in the name, we'll set up our `SendableChooser`s and actually display the options. Here's what to do:

- Initialize objects first:

```java
autoTypeChooser = new SendableChooser<>();
autoPositionChooser = new SendableChooser<>();
autoGoalChooser = new SendableChooser<>();
```

- actually connect the enum states to the choosers (using `setDefaultOption()` and `addOption()`) 
  - Here, we set default options as fall-throughs. Top would probably be our default starting position in a real competition for this game (easiest path for us to follow), so we set that accordingly.
  - The "Default Drive" option is for a basic driving auto moving the robot off the starting line--this is a backup in case any mechanisms are broken.

```java
autoTypeChooser = new SendableChooser<>();
autoPositionChooser = new SendableChooser<>();
autoGoalChooser = new SendableChooser<>();

autoTypeChooser.setDefaultOption("Segmented", AutoType.SEGMENTED);
autoTypeChooser.addOption("Trajectory", AutoType.TRAJECTORY);

autoPositionChooser.setDefaultOption("Top Start", AutoPosition.TOP);
autoPositionChooser.addOption("Middle Start", AutoPosition.MIDDLE);
autoPositionChooser.addOption("Bottom Start", AutoPosition.BOTTOM);

autoGoalChooser.setDefaultOption("Default Drive", AutoGoal.DEFAULT);
autoGoalChooser.addOption("Trench", AutoGoal.TRENCH);
autoGoalChooser.addOption("Generator Top", AutoGoal.GEN_TOP);
autoGoalChooser.addOption("Generator Bottom", AutoGoal.GEN_BOTTOM);

SmartDashboard.putData("Auto Type", autoTypeChooser);
SmartDashboard.putData("Auto Start Position", autoPositionChooser);
SmartDashboard.putData("Auto Goal", autoGoalChooser);
```

At the end, we display each `SendableChooser` on Shuffleboard with an appropriate label, just like any other constant or boolean. Remember to call `configureAutoChoosers()` in the constructor of `RobotContainer`, so that it is run when `RobotContainer` is instantiated.

---

Now that we've displayed the auto selection options, we need to actually return an auto `command` based off of the Shuffleboard input. In `getAutoCommand()`, we can read the selected options and act accordingly.

```java
AutoType type = autoTypeChooser.getSelected();
AutoPosition position = autoPositionChooser.getSelected();
AutoGoal goal = autoGoalChooser.getSelected();

if (type == AutoType.SEGMENTED) {
    switch (position) {
        case TOP:
            return new SegTopAuto(drivetrain, indexer, shooter, intake);
        case MIDDLE:
            return new SegMiddleAuto(drivetrain, indexer, shooter, intake);
        case BOTTOM:
            return new SegBottomAuto(drivetrain, indexer, shooter, intake);
    }
}

if (type == AutoType.TRAJECTORY) {
    if (position == AutoPosition.BOTTOM) return new TrajBottomAuto(drivetrain, indexer, shooter, intake);

    switch (goal) {
        case TRENCH:
            if (position == AutoPosition.TOP) return new TrajTopTrenchAuto(drivetrain, indexer, shooter, intake);
            else if (position == AutoPosition.MIDDLE) return new TrajMiddleTrenchAuto(drivetrain, indexer, shooter, intake);
        case GEN_TOP:
            if (position == AutoPosition.TOP) return new TrajTopGenTopAuto(drivetrain, indexer, shooter, intake);
            else if (position == AutoPosition.MIDDLE) return new TrajMiddleGenTopAuto(drivetrain, indexer, shooter, intake);
        case GEN_BOTTOM:
            if (position == AutoPosition.TOP) return new TrajTopGenBottomAuto(drivetrain, indexer, shooter, intake);
            else if (position == AutoPosition.MIDDLE) return new TrajMiddleGenBottomAuto(drivetrain, indexer, shooter, intake);
        case DEFAULT: // will go to drive distance command
    }
}

    return new DriveDistanceCommand(drivetrain, 2);
```

Here's a breakdown:

- The selected options are locally stored using `getSelected()` on each of the choosers.
- We then check the entered auto type (segmented vs. trajectory)
  - If it is segmented, we use a switch statement on the starting position to return the correct auto (since for the segmented autos, the ending position is always the same)
- We follow a similar process for the trajectory autos, instead using a switch on the `AutoGoal`. *Then*, within each `case` statement, we check the starting position, and return the right command.
  - If the starting position is bottom, we automatically return the bottom auto (it was decided that we wouldn't drive away from the power port if we started at the bottom, thus, no need to check the `AutoGoal`).
- Finally, if *nothing* is selected, or if the `DEFAULT` goal is selected for `AutoType.TRAJECTORY`, the default `DriveDistanceCommand` is returned. Once again, this acts as a fall-through in case of a Shuffleboard error or if the drive team forgets to click.

Thus, `RobotContainer.java` will look like this (imports/irrelevant content omitted for brevity):

```java
    ...

public class RobotContainer {

    ... // subsystems, notifier, etc

    private SendableChooser<AutoType> autoTypeChooser;
    private SendableChooser<AutoPosition> autoPositionChooser;
    private SendableChooser<AutoGoal> autoGoalChooser;

    public RobotContainer() {
        ... // set up our subsystems/default commands, register notifier, etc
        configureAutoChoosers();
    }

    private void configureButtonBindings() {
        ...
    }

    public void configureAutoChoosers() {
        autoTypeChooser = new SendableChooser<>();
        autoPositionChooser = new SendableChooser<>();
        autoGoalChooser = new SendableChooser<>();

        autoTypeChooser.setDefaultOption("Segmented", AutoType.SEGMENTED);
        autoTypeChooser.addOption("Trajectory", AutoType.TRAJECTORY);

        autoPositionChooser.setDefaultOption("Top Start", AutoPosition.TOP);
        autoPositionChooser.addOption("Middle Start", AutoPosition.MIDDLE);
        autoPositionChooser.addOption("Bottom Start", AutoPosition.BOTTOM);

        autoGoalChooser.setDefaultOption("Default Drive", AutoGoal.DEFAULT);
        autoGoalChooser.addOption("Trench", AutoGoal.TRENCH);
        autoGoalChooser.addOption("Generator Top", AutoGoal.GEN_TOP);
        autoGoalChooser.addOption("Generator Bottom", AutoGoal.GEN_BOTTOM);

        SmartDashboard.putData("Auto Type", autoTypeChooser);
        SmartDashboard.putData("Auto Start Position", autoPositionChooser);
        SmartDashboard.putData("Auto Goal", autoGoalChooser);
    }

    public Command getAutoCommand() {
         AutoType type = autoTypeChooser.getSelected();
         AutoPosition position = autoPositionChooser.getSelected();
         AutoGoal goal = autoGoalChooser.getSelected();

        if (type == AutoType.SEGMENTED) {
            switch (position) {
                case TOP:
                    return new SegTopAuto(drivetrain, indexer, shooter, intake);
                case MIDDLE:
                    return new SegMiddleAuto(drivetrain, indexer, shooter, intake);
                case BOTTOM:
                    return new SegBottomAuto(drivetrain, indexer, shooter, intake);
            }
        }

        if (type == AutoType.TRAJECTORY) {
            if (position == AutoPosition.BOTTOM) return new TrajBottomAuto(drivetrain, indexer, shooter, intake);

            switch (goal) {
                case TRENCH:
                    if (position == AutoPosition.TOP) return new TrajTopTrenchAuto(drivetrain, indexer, shooter, intake);
                    else if (position == AutoPosition.MIDDLE) return new TrajMiddleTrenchAuto(drivetrain, indexer, shooter, intake);
                case GEN_TOP:
                    if (position == AutoPosition.TOP) return new TrajTopGenTopAuto(drivetrain, indexer, shooter, intake);
                    else if (position == AutoPosition.MIDDLE) return new TrajMiddleGenTopAuto(drivetrain, indexer, shooter, intake);
                case GEN_BOTTOM:
                    if (position == AutoPosition.TOP) return new TrajTopGenBottomAuto(drivetrain, indexer, shooter, intake);
                    else if (position == AutoPosition.MIDDLE) return new TrajMiddleGenBottomAuto(drivetrain, indexer, shooter, intake);
                case DEFAULT: // will go to drive distance command
            }
        }

       return new DriveDistanceCommand(drivetrain, 2);
    }

    ...

}
```

## Execution

We're almost finished! All we have to do now is call the autos correctly in `Robot.java`:

```java
...

public class Robot extends TimedRobot {

    private RobotContainer robotContainer;
    private Command autoCommand;

    @Override
    public void robotInit() {
        robotContainer = new RobotContainer();
        Trajectories.setUpTrajectories();
    }

    @Override
    public void robotPeriodic() {
        CommandScheduler.getInstance().run();
        robotContainer.displayShuffleboard();
        if (SmartDashboard.getBoolean("Testing", false)) robotContainer.tuningPeriodic();
    }

    @Override
    public void autonomousInit() {
        autoCommand = robotContainer.getAutoCommand();

        if (autoCommand != null) {
            autoCommand.schedule();
        }

        robotContainer.resetIndexer();
    }

    @Override
    public void teleopInit() {
        if (autoCommand != null) {
            autoCommand.cancel();
        }

        robotContainer.resetIndexer();
    }

    ...
}
```

Here's what happens:

- First, we create a variable `Command autoCommand;` inside `Robot.java`.
- In `autonomousInit()`, we use `autoCommand = robotContainer.getAutoCommand();`
- If `autoCommand` is not `null` (which it never should be, since we always return either a selected or default command), we immediately schedule it with `autoCommand.schedule();`, thereby executing the auto.
- Finally, in `teleopInit()`, we check again whether autoCommand has a value; if it does, we call `autoCommand.cancel();` and move into teleop.

---

TODO: add screenshots of Shuffleboard choosers, just for visualization