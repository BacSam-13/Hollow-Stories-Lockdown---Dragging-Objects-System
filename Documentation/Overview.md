# Dragging Object System

The purpose of this project was to design a BluePrint that will be used as a base for different kinds of draggable objects, wich will be intended
to be used in different sections of the game.

To achieve this, we used the Gameplay Ability System from Unreal, and implemented new Gameplay Effects (GE), Gameplay Abilties (GA), and a Parent Class for a type of Box, that would grant Gameplay Effects.

# Implementation

First of all, we need to explain all the components used for this system:
- **BoxEffect**: A Box with a self-made parent class, designed to grant the GE to the player, and dicate the properties of the movement.
- **GE**: Gameplay Effect applied by the Box Effect, that while active, grants the GA required to move the object.
- **GA**: Gameplay Ability that allows the player to drag some objects, using the information granted by the BoxEffect, to dicate the direction of movement, and allowed axis.
- **StaticMesh**: Static Mesh, covered by a Box Collisions, that gives a visual of the object we are dragging, and delimit the area where the object can move.

Let´s break dowm every component, to explain how they work together.

## StaticMesh

The Statich Mesh was implemented as a simple cube, since we needed to work only with the X,Y axis, and we only required something basic.
This Static Mesh is also surrounded by a Box Collision, with the intention to help detect collisions with other objects in the enviorement, and stop it´s movement before the Static Mesh would go through some other mesh. This logic is also used with the self collisions in the player´s character.

On top of this, in front of every face with a normal perpendicular to the Z axis of the Statich Mesh, there is a Box Effect.

## BoxEffect

Every Box Effect is placed in front of one of the static mesh´s faces. And this has the purpose to grant the GE that will give the abilitie to the player to drag the object.
And to do so, it requires information that the Box Effect will grant with their parameters:

- uint8 **Move Direction**: Using the StaticMesh´s axis as reference, it tells us where is our new "front". The options are {Y+, Y-, X+, X-}, and this will be used to define our "NewForwardVector"
- bool **CanFrontBack**: Boolean that tells us the allowed axis to move.
- bool **CanRightLeft**: Boolean that tells us the allowed axis to move.

## GameplayEffect (GE)

This GameplayEffect is activated by the overlap of a BoxEffect, and has the duty to grant the GameplayAbilityy required to drag the objects.

## GameplayAbility (GA)

This GameplayAbility, uses the axis from the StaticMesh, along with the parametres given in the BoxEffect, to implement all the logic behing the movement of the object, by dragging it alosngside the level, next to the character.

This part is very complex, so we will explain it with full detail below.

# GA´s Logic

First of all, we will define the next elements that will be used during the explanation:

- **AC_DragObject**: BoxEffect, that is the source object for the GA.
- **DragActor**: The Blueprint that engloves all the components explained above.
- **Mesh**: Mesh for the Object we will be dragging.
- **BoxAux**: Auxiliar Box Collisions that surrounds the Mesh, and helps during the collision test.
- **NewForwardVector**: Vector that will tell us where is the new "front" for us while dragging the object.

Now, let´s continue with the explanation.

## Ability Activated

For this, we assume the player is inside a BoxEffect.
Once the ability is activated, we put the player into a proper position for the movement of dragging the object.

### Pseudo-Code

```text

```
