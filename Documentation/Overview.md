# Dragging Object System

The purpose of this project was to design a Blueprint that will be used as a base for different kinds of draggable objects, which will be intended to be used in different sections of the game.

To achieve this, we used the Gameplay Ability System from Unreal and implemented new Gameplay Effects (GE), Gameplay Abilities (GA), and a Parent Class for a type of Box that would grant Gameplay Effects.

# Implementation

First of all, we need to explain all the components used for this system:
- **BoxEffect**: A Box with a self-made parent class, designed to grant the GE to the player and dictate the properties of the movement.
- **GE**: Gameplay Effect applied by the Box Effect, which, while active, grants the GA required to move the object.
- **GA**: Gameplay Ability that allows the player to drag some objects, using the information granted by the BoxEffect to dictate the direction of movement and the allowed axes.
- **StaticMesh**: Static Mesh, covered by a Box Collision, that gives a visual representation of the object we are dragging and delimits the area where the object can move.

Let's break down every component to explain how they work together.

## StaticMesh

The Static Mesh was implemented as a simple cube, since we needed to work only with the X and Y axes, and we only required something basic.

This Static Mesh is also surrounded by a Box Collision, with the intention of helping detect collisions with other objects in the environment and stopping its movement before the Static Mesh goes through another mesh. This logic is also used with the self-collisions of the player's character.

On top of this, in front of every face with a normal perpendicular to the Z axis of the Static Mesh, there is a Box Effect.

## BoxEffect

Every Box Effect is placed in front of one of the Static Mesh's faces. This has the purpose of granting the GE that will give the player the ability to drag the object.

To do so, it requires information that the Box Effect will provide through its parameters:

- uint8 **Move Direction**: Using the Static Mesh's axes as a reference, it tells us where our new "front" is. The options are {Y+, Y-, X+, X-}, and this will be used to define our "NewForwardVector".
- bool **CanFrontBack**: Boolean that tells us which axes are allowed for movement.
- bool **CanRightLeft**: Boolean that tells us which axes are allowed for movement.

## GameplayEffect (GE)

This GameplayEffect is activated by the overlap of a BoxEffect and has the duty of granting the GameplayAbility required to drag the objects.

## GameplayAbility (GA)

This GameplayAbility uses the axes from the Static Mesh, along with the parameters given in the BoxEffect, to implement all the logic behind the movement of the object, by dragging it alongside the level, next to the character.

This part is very complex, so we will explain it in full detail below.

# GA's Logic

First of all, we will define the following elements that will be used during the explanation:

- **AC_DragObject**: BoxEffect, which is the source object for the GA.
- **DragActor**: The Blueprint that encompasses all the components explained above.
- **Mesh**: Mesh for the object we will be dragging.
- **BoxAux**: Auxiliary Box Collisions that surround the Mesh and help during the collision test.
- **NewForwardVector**: Vector that will tell us where the new "front" is for us while dragging the object.
- **MC**: The Main Character used by the player.

Now, let's continue with the explanation.

## Ability Activated

For this, we assume the player is inside a BoxEffect.

Once the ability is activated, we put the player into a proper position for the movement required to drag the object.

### Pseudo-Code

```text
GET NewForwardVector = Normalize(AC_DragObject.NewForwardVector) // Vector for our new Forward Direction
SET NewRightwardVector = NewForwardVector x (0,0,-1) // Vector for our new Rightward Direction

// We update the character and control rotation to match the DragActor's direction of movement
SET NewRotationMC = MakeRotationFromAxes(NewForwardVector, NewRightwardVector, (0,0,1)) // New Rotation for the MC
MC.SetActorRotation(NewRotationMC)
MC.UseControllerRotationYaw = true
MC.Controller.SetControlRotaion(NewRotationMC)

Attach DragActor TO MC
```

Now, during every tick that the Ability is activated, we will do what is described below.

```text
MC.SetActorRotation(NewRotationMC)
AngForward, AngRightward = GetAngles() // We get the original movement inputs, transforming them into the new rotation. (Will be explained below)

SET Speed = GetMaxSpeed(MC) // MC's maximum movement speed (can be changed to another value)

// We analyze the new angles of movement and check if the movement is allowed, to move the MC in the proper direction with its respective speed
// If the movement is not allowed by the booleans Can_Right_Left or Can_Front_Back, the ability is ended

// Check Front/Back movement
IF Can_Front_Back == TRUE:

    IF AngForward < 45:
        Movement(IsForward/Backward = TRUE, DirectionSpeed = Speed)

    ELSE IF AngForward > 135:
        Movement(IsForward/Backward = TRUE, DirectionSpeed = -Speed)

ELSE:
    IF AngForward < 45 OR AngForward > 135:
        EndAbility()
END IF

// Check Right/Left movement
IF Can_Right_Left == TRUE:

    IF AngRightward < 45:
        Movement(IsForward/Backward = FALSE, DirectionSpeed = Speed)

    ELSE IF AngRightward > 135:
        Movement(IsForward/Backward = FALSE, DirectionSpeed = -Speed)

ELSE:
    IF AngRightward < 45 OR AngRightward > 135:
        EndAbility()
END IF
```

Finally, we will explain the functions used above, including Movement(), which will move the MC along with the DragActor, and CheckCollision(), which is used in Movement() to check if a movement is allowed and does not cause a collision.

```text
FUNCTION GetAngles():
    SET Input = MakeVector(MC.CurrentMoveInput, 0) // Make it (x,y,0)
    Set NewVector = RotateVector(Input, MatrixRotation(Forward(0,1,0), Right(1,0,0), Up(0,0,1) ) )
    
    NewVector = RotateVector(NewVector, NewRotationMC)
    SET InputVector = Normalize(NewVector)
    
    SET AngForward = cos^{-1} = (NewForwardVector * InputVector)
    SET AngRightward = cos^{-1} = (NewRightwardVector * InputVector)
    
    RETURN AngForward, AngRightward
```

```text
DEFAULT VALUES: // Keep the block state on the movement axes
    BlockForward = true
    BlockRightward = true

FUNCTION Movement(IsForward/Backward, DirectionSpeed):
    IF IsForward/Backward:
        SET ActualBlock = BlockForward
    ELSE:
        SET ActualBlock = BlockRightward
    END IF

    // Select movement direction
    IF Is_Forward/Backward == TRUE:
        DirectionVector = New_ForwardVector
    ELSE:
        DirectionVector = New_RightwardVector
    END IF
    
    // Calculate target location
    MovementVector = DirectionVector * DirectionSpeed
    TargetLocation = DragActor.Location + MovementVector
    
    // Calculate interpolation speed
    CurrentVelocity = GetCharacterMovementVelocity()
    CurrentSpeed = GetVectorLength(CurrentVelocity)
    
    IF CurrentSpeed == 0:
        InterpSpeed = 8.33335 // Min Speed so we don't divide by zero
    ELSE:
        InterpSpeed = CurrentSpeed
    END IF
    
    InterpSpeed = InterpSpeed / Speed
    
    // Smoothly interpolate toward the target location
    NewLocation = VInterpTo(
        CurrentLocation = DragActor.Location,
        TargetLocation = TargetLocation,
        DeltaTime = GetWorldDeltaSeconds(),
        InterpSpeed = InterpSpeed
    )
    
    // Check whether the new location causes a collision
    IsCollision = CheckCollision(DragActor, NewLocation)
    
    IF IsCollision == TRUE AND Actual_Block == TRUE:
    
        // Update the corresponding movement block
        IF Is_Forward/Backward == TRUE:
            Block_Rightward = FALSE
        ELSE:
            Block_Forward = FALSE
        END IF
    
    ELSE:
    
        // Apply movement input to the character
        MC.AddMovementInput( WorldDirection = DirectionVector, ScaleValue = 1.0 )
    
        // Update the corresponding movement block
        IF Is_Forward/Backward == TRUE:
            Block_Rightward = TRUE
        ELSE:
            Block_Forward = TRUE
        END IF
    END IF
    
    // Update the actual movement block state
    IF Is_Forward/Backward == TRUE:
        Actual_Block = Block_Forward
    ELSE:
        Actual_Block = Block_Rightward
    END IF
```

```text
FUNCTION CheckCollision(DragActor, NewLocation):
    
    // First collision check: Box Trace
    BoxCollision = BoxAux.BoxCollision

    // Calculate the predicted box position
    MovementOffset = NewLocation - DragActor.Location

    BoxStart = BoxCollision.Location
    BoxEnd = BoxCollision.Location + MovementOffset

    BoxHalfSize = GetScaledBoxExtent(BoxCollision)
    BoxRotation = GetWorldRotation(BoxCollision)

    BoxHit = BoxTraceSingleForObjects(
        Start = BoxStart,
        End = BoxEnd,
        HalfSize = BoxHalfSize,
        Orientation = BoxRotation,
        ObjectTypes = [WorldStatic],
        IgnoreSelf = TRUE
    )



    // Second collision check: Capsule Trace (We will assume the MC uses a Capsule as a Collision)
    Capsule = GetCapsuleComponent(MC)

    // Calculate the predicted capsule position
    MovementOffset = NewLocation - DragActor.Location

    CapsuleStart = Capsule.Location
    CapsuleEnd = Capsule.Location + MovementOffset

    CapsuleRadius = GetScaledCapsuleRadius(Capsule)
    CapsuleHalfHeight = GetScaledCapsuleHalfHeight(Capsule)

    CapsuleHit = CapsuleTraceMultiForObjects(
        Start = CapsuleStart,
        End = CapsuleEnd,
        Radius = CapsuleRadius,
        HalfHeight = CapsuleHalfHeight,
        ObjectTypes = [WorldStatic],
        ActorsToIgnore = [DragActor],
        IgnoreSelf = TRUE
    )


    // Combine collision results
    ThereIsCollision = BoxHit OR CapsuleHit

    RETURN ThereIsCollision
```

# Conclusion

Thanks to this complex system, we are able to grant an ability that allows the player to attach themselves to an object and drag it alongside the level, checking for collisions and updating the input movement to match the object's movement, while ignoring any changes caused by the original input system.

Note: At the moment of writing this, the animations for the ability haven´t been completed. They wil be completed soon by it´s corresponding departament, and added into the logic of the system.
