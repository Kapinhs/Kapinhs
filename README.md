#include maps/mp/_utility;
#include common_scripts/utility;
#include maps/mp/zombies/_zm;
#include maps/mp/zombies/_zm_perks;
#include maps/mp/zombies/_zm_utility;
#include maps/mp/gametypes_zm/_hud_util;
#include maps/mp/gametypes_zm/_hud_message;
init()
{
precacheshader( "damage_feedback" );
precacheshader( "menu_mp_fileshare_custom" );
level.perk_purchase_limit = getdvarintdefault( "cmPlayerPerkLimit", 9 );
level.cmperkdoubletapfirerate = getdvarfloatdefault( "cmPerkDoubleTapFireRate", 0,5 );
setdvar( "perk_weapRateMultiplier", level.cmperkdoubletapfirerate );
perk_machine_removal( "specialty_rof" );
perk_machine_removal( "specialty_additionalprimaryweapon" );
perk_machine_removal( "specialty_flakjacket" );
perk_machine_removal( "specialty_nomotionsensor" );
level._random_zombie_perk_cost = undefined;
level._challenges = undefined;
setdvar( "player_strafeSpeedScale", 1 );
setdvar( "player_sprintStrafeSpeedScale", 1 );
setdvar( "player_backSpeedScale", 1 );
setdvar( "jump_slowdownEnable", 0 );
level thread onplayerconnect();
}
onplayerconnect()
{
for(;;)
{
level waittill( "connected", player );
player iprintln( "^1Cold War Zombies" );
player thread zombie_health();
player thread visuals();
player thread onplayerspawned();
}
}
zombie_health()
{
level endon( "end_game" );
self endon( "disconnect" );
for(;;)
{
level waittill( "start_of_round" );
if( level.zombie_health > 10000 )
{
level.zombie_health = 10000;
}
wait 0,05;
}
}
visuals()
{
self setclientdvar( "r_fog", 0 );
self setclientdvar( "r_dof_enable", 0 );
self setclientdvar( "r_lodBiasRigid", -1000 );
self setclientdvar( "r_lodBiasSkinned", -1000 );
self setclientdvar( "r_lodScaleRigid", 1 );
self setclientdvar( "r_lodScaleSkinned", 1 );
self useservervisionset( 1 );
self setvisionsetforplayer( "remote_mortar_enhanced", 0 );
}
onplayerspawned()
{
level endon( "end_game" );
self endon( "disconnect" );
self waittill( "spawned_player" );
self setperk( "specialty_unlimitedsprint" );
self thread drop();
self thread rof();
self thread rof_ready();
self thread quick_revive();
self thread quick_revive_ready();
self thread staminup();
self thread health_bar_hud();
}
drop()
{
level endon( "end_game" );
self endon( "disconnect" );
if( self meleebuttonpressed() )
{
duration = 0;
while( self meleebuttonpressed() )
{
duration = duration + 1;
if( duration == 25 )
{
weap = self getcurrentweapon();
self dropitem( weap );
break;
}
wait 0,05;
}
}
wait 0,05;
?;//Jump here. This may be a loop, else, continue, or break. Please fix this code section to re-compile.
}
rof()
{
level endon( "end_game" );
self endon( "disconnect" );
rof_hud = newclienthudelem( self );
rof_hud.alignx = "center";
rof_hud.aligny = "bottom";
rof_hud.horzalign = "user_center";
rof_hud.vertalign = "user_bottom";
rof_hud.y = rof_hud.y - 35;
rof_hud.alpha = 0;
rof_hud.color = ( 1, 1, 1 );
rof_hud.hidewheninmenu = 1;
rof_hud setshader( "menu_mp_fileshare_custom", 32, 32 );
self waittill_any( "perk_acquired", "perk_lost" );
for(;;)
{
while( self getvelocity() >= 1 && self.perks_active.size >= 3 )
{
duration = 0;
rof_hud.alpha = 0;
self unsetperk( "specialty_rof" );
while( self getvelocity() == 0 )
{
duration = duration + 1;
if( duration >= 100 )
{
rof_hud.alpha = 1;
self setperk( "specialty_rof" );
}
wait 0,05;
}
}
rof_hud.alpha = 0;
self unsetperk( "specialty_rof" );
wait 0,05;
}
}
rof_ready()
{
level endon( "end_game" );
self endon( "disconnect" );
rof_ready_hud = newclienthudelem( self );
rof_ready_hud.alignx = "right";
rof_ready_hud.aligny = "bottom";
rof_ready_hud.horzalign = "user_right";
rof_ready_hud.vertalign = "user_bottom";
rof_ready_hud.x = rof_ready_hud.x - 155;
rof_ready_hud.alpha = 0;
rof_ready_hud.color = ( 1, 1, 1 );
rof_ready_hud.hidewheninmenu = 1;
rof_ready_hud setshader( "specialty_doubletap_zombies", 32, 32 );
self waittill_any( "perk_acquired", "perk_lost" );
for(;;)
{
if( self.perks_active.size >= 3 )
{
rof_ready_hud.alpha = 1;
}
else
{
rof_ready_hud.alpha = 0;
}
wait 0,05;
}
}
quick_revive()
{
level endon( "end_game" );
self endon( "disconnect" );
for(;;)
{
if( self.health < self.maxhealth && self hasperk( "specialty_quickrevive" ) )
{
self.health = self.health + 1;
}
wait 0,1;
}
}
quick_revive_ready()
{
level endon( "end_game" );
self endon( "disconnect" );
qr_hud = newclienthudelem( self );
qr_hud.alignx = "left";
qr_hud.aligny = "bottom";
qr_hud.horzalign = "user_left";
qr_hud.vertalign = "user_bottom";
qr_hud.x = qr_hud.x + 155;
qr_hud.alpha = 0;
qr_hud.color = ( 1, 1, 1 );
qr_hud.hidewheninmenu = 1;
qr_hud setshader( "damage_feedback", 32, 32 );
self waittill_any( "perk_acquired", "perk_lost" );
for(;;)
{
if( getplayers().size <= 1 && self hasperk( "specialty_quickrevive" ) )
{
qr_hud.alpha = 1;
}
else
{
qr_hud.alpha = 0;
}
wait 0,05;
}
}
staminup()
{
level endon( "end_game" );
self endon( "disconnect" );
for(;;)
{
self waittill_any( "perk_acquired", "perk_lost" );
if( self hasperk( "specialty_longersprint" ) )
{
self setperk( "specialty_movefaster" );
self setperk( "specialty_fallheight" );
}
else
{
self unsetperk( "specialty_movefaster" );
self unsetperk( "specialty_fallheight" );
}
}
}
health_bar_hud()
{
level endon( "end_game" );
self endon( "disconnect" );
flag_wait( "initial_blackscreen_passed" );
health_bar = self createprimaryprogressbar();
if( level.script == "zm_buried" || level.script == "zm_tomb" )
{
health_bar setpoint( undefined, "BOTTOM", -360, -95 );
}
else
{
health_bar setpoint( undefined, "BOTTOM", -360, -70 );
}
health_bar.hidewheninmenu = 1;
health_bar.bar.hidewheninmenu = 1;
health_bar.barframe.hidewheninmenu = 1;
health_bar_text = self createprimaryprogressbartext();
if( level.script == "zm_buried" || level.script == "zm_tomb" )
{
health_bar_text setpoint( undefined, "BOTTOM", -285, -95 );
}
else
{
health_bar_text setpoint( undefined, "BOTTOM", -285, -70 );
}
health_bar_text.hidewheninmenu = 1;
while( 1 )
{
if( IsDefined( self.e_afterlife_corpse ) )
{
if( health_bar.alpha != 0 )
{
health_bar.alpha = 0;
health_bar.bar.alpha = 0;
health_bar.barframe.alpha = 0;
health_bar_text.alpha = 0;
}
wait 0,05;
continue;
}
if( health_bar.alpha != 1 )
{
health_bar.alpha = 1;
health_bar.bar.alpha = 1;
health_bar.barframe.alpha = 1;
health_bar_text.alpha = 1;
}
health_bar updatebar( self.health / self.maxhealth );
health_bar_text setvalue( self.health );
wait 0,05;
}
}
