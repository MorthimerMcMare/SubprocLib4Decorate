# SubprocLib4Decorate
A simple library written on ZScript which allows use a states parallelizing and subroutines in Decorate.

A [main branch](https://github.com/MorthimerMcMare/SubprocLib4Decorate) is compatible with anything from GZDoom 2.4.0. A [new engine versions branch](https://github.com/MorthimerMcMare/SubprocLib4Decorate/tree/new_engine_versions) tries to provide more convenient API, but requires at least GZDoom 4.3.1.

## API

### GZDoom 2.4.0

To inherit not only from Actor class, you should copy necessary functions [from the base "SubprocActor" class](https://github.com/MorthimerMcMare/SubprocLib4Decorate/blob/compatibility/ZScript.zsc#L9) to some ZScript file. For example, if you want to use only a parallelization for the DoomImp replacement:

```Csharp
// ZScript:
class DoomImpParallelExample: DoomImp {
	action void PAR_Thread( StateLabel label, EParallelExecDirectives execparams = PARSPR_SkipTNT1 ) {
		SubproclibParallelExecKeeper.GetKeeper( invoker ).AddThread( label, execparams );
	}
	action void PAR_Stop( StateLabel label = NULL ) {
		SubproclibParallelExecKeeper.GetKeeper( invoker ).RemoveThread( label );
	}
	action void PAR_FrozeFlow( StateLabel label, bool freezethread ) {
		SubproclibParallelExecKeeper.GetKeeper( invoker ).ThreadFrozeControl( label, freezethread );
	}
}
```

```Csharp
// Decorate:
Actor EvinceImp: DoomImpParallelExample {
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


### GZDoom 4.3.1

Just use a predefined Mixin in some ZScript file:
```Csharp
// ZScript:
class DoomImpParallelExample: DoomImp {
	Mixin Parallelizing;
}
```

So, decorate example is same.
