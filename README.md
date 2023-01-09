# SubprocLib4Decorate
A simple library written on ZScript which allows use a states parallelizing and subroutines in Decorate.

A [main branch](https://github.com/MorthimerMcMare/SubprocLib4Decorate) is compatible with anything starting from GZDoom 2.4.0. A [new engine versions branch](https://github.com/MorthimerMcMare/SubprocLib4Decorate/tree/new_engine_versions) tries to provide more convenient API, but requires at least GZDoom 4.3.1.

If you find a bug, please report about it to me. For example, [here](https://github.com/MorthimerMcMare/SubprocLib4Decorate/issues).

Note that this is a beta-version of the library.


<p><br></p>

## API

See [an "/Examples/" folder](https://github.com/MorthimerMcMare/SubprocLib4Decorate/tree/new_engine_versions/Examples) for them.

### Subroutines

```
SUBP_Call( State subproc )
```
Saves current state position and jumps to the specified `state` (by name or offset). This function also may be nested to and called from other subroutines, but addresses stack is limited at depth of 255.

```
SUBP_Return()
```
Returns to the state on the top of the addresses stack. Cannot be called if current stack size is 0.


```
SUBP_InitHandler()
SUBP_RemoveHandler()
```
Internal functions, initializes and deletes a subroutines handler. May be used for temporary disabling the subroutines, but I advice to avoid this codepointers.


<p><br></p>

### Parallelizing

All threads terminate when actor is gone.

```
PAR_Thread( State startstate[, int steps[, int params]] )
```

Starts a parallel thread on current actor. `steps` defines a countdown (in states with non-zero delay) to terminate this thread; by default equals to `PARSTEPS_Infinite` (-1). `params` defines behaviour of this thread and may use next constants:

* `PARSPR_All` or `PARSPR_Any` to copy all of the sprites/frames, including "TNT1A0";

* `PARSPR_SkipTNT1`, `PARSPR_SkipTNT1A0`, `PARSPR_IgnoreTNT1` or `PARSPR_IgnoreTNT1A0` to apply all sprites/frames excluding "TNT1A0" (default behaviour);

* `PARSPR_None`, `PARSPR_IgnoreAll` or `PARSPR_SkipAll` to only execute a functions in a thread.

* Also you may add a flag `PAREXEC_NoFunctionsCall` via `|`; it will block any codepointers in the thread from executing (...including "`PAR_Stop()`", yep).

```
PAR_Stop( [State threadid] )
```
Stops the current thread, if argument is omitted, or else the thread specified by its first state.

Note: when thread reaches [a "Stop" keyword](https://zdoom.org/wiki/Actor_states#Flow_control) or disappears in any other manner, main actor destroys, too. So writing a "`TNT1 A 0 PAR_Stop`" may be a bad idea; better set larger delay, like "`TNT1 A 1 PAR_Stop`", or use a `steps` argument in the "`PAR_Thread()`".


<p><br></p>

## Engine version differences

### GZDoom 4.3.1

Just use a predefined Mixin in some ZScript file:
```Csharp
// ZScript:
class DoomImpParallelBase: DoomImp {
	Mixin Parallelizing;
}
```

### GZDoom 2.4.0

To inherit not only from Actor class, you should copy necessary functions [from the base "SubprocActor" class](https://github.com/MorthimerMcMare/SubprocLib4Decorate/blob/compatibility/ZScript.zsc#L11) to some ZScript file. For example, if you want to use only a parallelization for the DoomImp replacement:

```Csharp
// ZScript:
class DoomImpParallelExample: DoomImp {
	action void PAR_Thread( StateLabel label, int steps = PARSTEPS_Infinite, EParallelExecDirectives execparams = PARSPR_SkipTNT1 ) {
		SubproclibParallelExecKeeper.GetKeeper( invoker ).AddThread( label, steps, execparams );
	}
	action void PAR_Stop( StateLabel label = NULL ) {
		SubproclibParallelExecKeeper.GetKeeper( invoker ).RemoveThread( label );
	}
	action void PAR_FrozeFlow( StateLabel label, bool freezethread ) {
		SubproclibParallelExecKeeper.GetKeeper( invoker ).ThreadFrozeControl( label, freezethread );
	}
}
```

### Decorate

Same for both cases.

```Csharp
// Decorate:
Actor EvinceImp: DoomImpParallelBase {
	Health 75

	States {
	Missile:
		TROO E 0 PAR_Thread( "Rotating" )
		TROO E 0 PAR_Thread( "MissileFireSpawn" )
	Melee:
		TROO EF 8 A_FaceTarget
		TROO G 6 A_TroopAttack 
		Goto See
	Rotating:
		TNT1 AAAAAAAAAAAAAAA 1 A_SetAngle( angle + 11.25 )
		TNT1 A 2 A_FaceTarget
		TNT1 A 0 PAR_Stop( "MissileFireSpawn" )
		TNT1 A 1 PAR_Stop    // "PAR_Stop()" without arguments stops current thread.
		Stop
	Pain:
		TROO H 2
		TROO H 2 A_Pain
		Goto See
	Death:
		TROO I 0 PAR_Stop( "MissileFireSpawn" )
		Goto Super::Death
	XDeath:
		TROO N 0 PAR_Stop( "MissileFireSpawn" )
		Goto Super::XDeath
	MissileFireSpawn:
		TNT1 A 2 A_SpawnItemEx( "SpawnFire", -radius, FRandom( -96.0, 96.0 ) )
		Loop
	}
}
```
