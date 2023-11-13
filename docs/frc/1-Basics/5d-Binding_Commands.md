# Binding Commands (Roller Intake)

Now that we have created the commands, we need a way to actually call/use them. This is where bindings come in. All of the commands are bound to certain buttons/triggers on an Xbox controller so we can control the robot.

Button bindings are created in the file `RobotContainer.java.` Certain parts in this file will be omitted for brevity, so read other lessons to get a full picture of what `RobotContainer` entails.

Here is the flowchart explaining the entire program again. If the flowchart does not load, please [press here](https://drive.google.com/file/d/1OdYeyfamvG7weoWkQY1DDX4__NVKelgm/view?usp=sharing) to view a copy of it.

<iframe frameborder="0" style="width:100%;height:343px;" src="https://app.diagrams.net/?lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Roller%20Intake%20Flowchart.drawio#Uhttps%3A%2F%2Fdrive.google.com%2Fuc%3Fid%3D1OdYeyfamvG7weoWkQY1DDX4__NVKelgm%26export%3Ddownload"></iframe>

For our flowchart, this section corresponds to the red bottom left section of the flowchart, where we check what button is pressed and schedule the appropriate command.

## Robot Container Format

Below is the basic format of the `RobotContainer` file:

```java
package frc.robot;

import frc.robot.commands.rollerintake.RollerIntakeNeutralCommand;
import frc.robot.commands.rollerintake.IntakeIntakeNeutralCommand;
import frc.robot.commands.rollerintake.EjectIntakeNeutralCommand;

import edu.wpi.first.wpilibj.Notifier;
import edu.wpi.first.wpilibj.smartdashboard.SmartDashboard;
import edu.wpi.first.wpilibj2.command.Command;
import frc.robot.subsystems.SnailSubsystem;
import frc.robot.util.SnailController;
import edu.wpi.first.wpilibj.XboxController.Button;

import java.util.ArrayList;

import static frc.robot.Constants.ElectricalLayout.CONTROLLER_DRIVER_ID;
import static frc.robot.Constants.ElectricalLayout.CONTROLLER_OPERATOR_ID;
import static frc.robot.Constants.UPDATE_PERIOD;;

public class RobotContainer {

    private SnailController driveController;
    private SnailController operatorController;

    private ArrayList<SnailSubsystem> subsystems;

    public RobotContainer() {
        driveController = new SnailController(CONTROLLER_DRIVER_ID);
        operatorController = new SnailController(CONTROLLER_OPERATOR_ID);

        configureSubsystems();
        configureButtonBindings();
    }

    /**
     * Declare all of our subsystems and their default bindings
     */
    private void configureSubsystems() {
        // declare each of the subsystems here

        subsystems = new ArrayList<>();
        // add each of the subsystems to the arraylist here
    }

    /**
     * Define button -> command mappings.
     */
    private void configureButtonBindings() {

    }

    ...
}
```

### RobotContainer()

This is the constructor for the class. The `SnailController`s, the subsystems, and their default commands are defined here. The main purpose of the constructor is just to call some of our other subfunctions which will each do different things.

### configureSubsystems()

This function is where we will create instances of all of our subsystems and define some things about them such as default commands. There is some extra code here, but you don't have to worry too much about it for now. We will mainly work in this function and the next one.

### configureButtonBindings()

This function is where all the button bindings actually happen, and is where we'll mainly work alongside the previous function in this lesson. In other words, we add functionality such that a button press will call a specific command.

### Other Functions

There are more functions, but we will cover them later when we go over advanced topics such as diagnostic data and autonomous mode.

## Adding Functionality

### The Constructor

```java
    ...

    private final SnailController driveController;
    private final SnailController operatorController;

    private final RollerIntake rollerIntake;

    ...

    private void configureSubsystems() {
        rollerIntake = new RollerIntake(); // NEW
        rollerIntake.setDefaultCommand(new RollerIntakeNeutralCommand(rollerIntake)); // NEW

        subsystems = new ArrayList<>();

        subsystems.add(rollerIntake);
    }

```

* First, we create a new `RollerIntake` object and set assign it to `rollerIntake`.
* Then, the "default command" for the `RollerIntake` is set. A default command is a command that runs for a subsystem when *no other* command is scheduled. Since we do not want the robot to do anything until we press a button, we set the default command to the `RollerIntakeNeutralCommand` (having the intake run at a low, constant speed).
* Finally, we "register" our subsystem within our robot by adding it to the `ArrayList`. An `ArrayList` is essentially a container that can grow to store as many elements as we want in it.

### Binding Commands

```java
private void configureButtonBindings() {
    operatorController.getButton(Button.kX.value).whileActiveOnce(new RollerIntakeEjectCommand(rollerIntake));
    operatorController.getButton(Button.kA.value).whileActiveOnce(new RollerIntakeIntakeCommand(rollerIntake));
}
```

Here, we're taking our operator XBox controller and setting the X button is being set to run `RollerIntakeEjectCommand`, while the A button is being set to run `RollerIntakeIntakeCommand`. We will break this down further below.

#### getButton()

* When this function is run, it returns a [JoystickButton](https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj2/command/button/JoystickButton.html) object that corresponds to a physical button on the controller. The format for the parameter is `Button.k(button).value`, where "button" is the chosen button. In the first case it would return the X button, and the A button for the second. (You will probably need an `import` statement for the `Button` class).
* Now that a `JoystickButton` object has been returned, a multitude of functions can be used to actually bind a command to certain actions.

#### Button Binding Functions

There are many, many functions you can choose from, but we will go over a few of them here.

**`whenPressed()`** - When the button is pressed, the command is run once and then it reverts to the default command immediately. This is useful for quick actions that we do not want to run for a period of time.

**`whenReleased()`** - When the button is released, the command is run once and then it reverts to the default command immediately. This is also useful for quick actions that we do not want to run for a period of time.

**`whileActive()`** - Runs the command constantly as the button is held down. It will constantly schedule the command, but stops when the button is let go. We typically don't use this and instead choose to use the next option (`whileActiveOnce()`).

**`whileActiveOnce()`** - Runs the command when the button is first held down and will continue running for as long as the button stays down. If the command ends while the button is still active, it will *not* be rescheduled. When the button itself is let go, the command will be cancelled and the default command will begin running.

**`toggleWhenPressed()`** - This function will toggle the command to constantly be run when the button is first pressed. When the button is released, the command will continue running until the button is pressed again and toggles it to stop running.

> [!WARNING]
> There is also a function in our controller to use Xbox triggers. Triggers unfortunately have a bit of a different naming convention, so see the below link for details on that)

See [here](https://docs.wpilib.org/en/latest/docs/software/commandbased/binding-commands-to-triggers.html) for more controller/joystick functions from WPILib.

#### The Function We Chose

Since we want the motors to spin as long as the button is pressed, we use the `whileActiveOnce()` function. Once again, since we return false in `isFinished()`, we choose `whileActiveOnce()`.

Inside of the parentheses is the argument, which is the command we want to use. We use the `new` keyword here to create a quick object from our command, since we need to pass an object into the function.

---

Here is all of the code covered in this lesson.

```java
package frc.robot;

import frc.robot.commands.rollerintake.RollerIntakeNeutralCommand;
import frc.robot.commands.rollerintake.RollerIntakeIntakeCommand;
import frc.robot.commands.rollerintake.RollerIntakeEjectCommand;

import edu.wpi.first.wpilibj.Notifier;
import edu.wpi.first.wpilibj.smartdashboard.SmartDashboard;
import edu.wpi.first.wpilibj2.command.Command;
import frc.robot.subsystems.SnailSubsystem;
import frc.robot.util.SnailController;
import frc.robot.subsystems.RollerIntake;
import edu.wpi.first.wpilibj.XboxController.Button;

import java.util.ArrayList;

import static frc.robot.Constants.ElectricalLayout.CONTROLLER_DRIVER_ID;
import static frc.robot.Constants.ElectricalLayout.CONTROLLER_OPERATOR_ID;
import static frc.robot.Constants.UPDATE_PERIOD;

public class RobotContainer {

    private final SnailController driveController;
    private final SnailController operatorController;

    private final RollerIntake rollerIntake;

    private ArrayList<SnailSubsystem> subsystems;

    public RobotContainer() {
        driveController = new SnailController(CONTROLLER_DRIVER_ID);
        operatorController = new SnailController(CONTROLLER_OPERATOR_ID);

        configureSubsystems();
        configureButtonBindings();
    }

    private void configureSubsystems() {

        rollerIntake = new RollerIntake();
        rollerIntake.setDefaultCommand(new RollerIntakeNeutralCommand(rollerIntake));

        subsystems = new ArrayList<>();

        subsystems.add(rollerIntake)
    }

    private void configureButtonBindings() {
        operatorController.getButton(Button.kX.value).whileActiveOnce(new RollerIntakeEjectCommand(rollerIntake));
        operatorController.getButton(Button.kA.value).whileActiveOnce(new RollerIntakeIntakeCommand(rollerIntake));
    }

    ...
}
```

## Final Remarks

With that, we're done! We've created a fully functional subsystem with bound commands. Hopefully you can try this out on a robot and see the results yourself. Click [here](https://github.com/FRC1257/training-programs/tree/master/basics/roller-intake) for a link to the full code.

This is one of the first and most basic subsystems we can program, but you can be sure that we will learn how to program much, much more! We're going to go over a few more common subsystems, starting with another style of intake: a claw intake.
