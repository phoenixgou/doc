Shooting in SpatialOS
ally@
Challenges
Server Authority
Workers
Communication
Shooting
Reloading
Schema Components
Challenges

Worker boundaries
Scaling the implementation to work for huge worlds
Cheating
Preventing users gaining significant advantage through hacking
Efficiency
Minimizing overhead for achieving functionality
Server Authority

Players have server authoritative shooting to prevent Cheating, this covers:

A Player shoots and hits a player when they shouldn’t be able to (due to distance or obstacles)

A Player having infinite ammunition (and never needing to reload)

Matching the RPCs available to clients (shooting and reloading)

Workers 

There will be a maximum of 4 workers involved.

Each Player will be controlled by a local client and a cloud server. 

The cloud servers can be either the same server or different (where Player 1 shoots Player 2 across a worker boundary). 

Communication
Shooting
Player 1 Shoot event
Tell nearby players to show Player 1’s gun being fired

Player 1 Attempt Shot command
Tell the server we want to damage Player 2

Player 2 Hit command
The Player 1 Server tells the Player 2 Server that Player 2 was hit

Player 2 Took Damage event
Tell Player 2 client to lower the health bar
Tell other players we were hit (for example, in the leg)


N.B. If the same cloud server controls both Player 1 and Player 2 then all the communication between the servers in the diagram will short circuit and avoid messages across the network!
Reloading

Player 1 Reload event
Tell other players to play our reloading animation

Player 1 Attempt Reload command
Tell the server we want to reload


Schema Components

Client Shooting 
Client Authoritative

Used by shooter client to trigger an event whenever the player shoots.
package improbable.shooting;

type ShootDetails {}

component ClientShooting {
   id = 10000;
   event ShootDetails shoot;
}




Server Shooting 
Server Authoritative

Used by shooter to track ammunition count, preventing shooting without ammo and handling reloading.
package improbable.shooting;

type ShotAttemptDetails {
	option<EntityId> target = 1;
	// whatever validation details you want	
}

type ReloadAttemptDetails {}

type ValidationResponse {
	bool success = 1;
}

component ServerShooting {
  id = 10001;
  int32 ammo = 1;
  command ValidationResponse attempt_shot(ShotAttemptDetails);
  command ValidationResponse attempt_reload(ReloadAttemptDetails);
}


Can Be Shot 
Server Authoritative

Used by target getting shot to receive commands from shooter and broadcast events about taking damage.
package improbable.shooting;

type HitDetails {
	int32 damage = 1;
}

type HitResponse {
	bool success = 1;
}

component CanBeShot {
  id = 10002;
  event HitDetails took_damage;
  command HitResponse hit(HitDetails);
}

