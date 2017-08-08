---
layout: post
title:  "Strafing Theory by injx"
date:   2017-08-08 00:00:00
categories: DeFRaG strafing theory quake
---

This is a post that was of great use to me written by injx on funender, it seems they are no longer hosting it however. Re-uploading for archival purposes.

## Introduction
This article will attempt to explain why certain angles have to be used when strafe-jumping, and why strafe-jumping is necessary in the first place. It is aimed at those who already have a practical knowledge of strafe-jumping, and wish to know more about what is really going on. Although this article is based on Quake 3, the theory may be applied to many games, especially those based on Quake engines.


If you enjoyed this article, you may be interested in its sequel, Circle-Jump Theory **COMING SOON**


Firstly, you should know that there is one important difference between standing on a surface and being airborne, that is a surface has friction. Therefore if you are in contact with a surface and you stop providing movement (i.e. release all keys) then the frictional force will decelerate your movement to zero. In the air there is no friction, and so as long as you are not in contact with another object, when you release all keys your speed will remain constant.


In-game, a player has control of two key things: their orientation, and the movement keys being pressed. The movement keys can provide acceleration; the combination of movement keys, and the orientation of the player, determines which direction the player wishes this acceleration to be applied in.


For example, by holding Forwards and Right Strafe, the player's acceleration vector, **a**, looks like this:


<img src="{{ "/images/strafe_theory/acceleration.gif" | prepend: site.baseurl }}" alt="Acceleration vectors">


It is this acceleration, provided by the movement keys, which gives the player a change in velocity, which ultimately changes his position.


However, the Quake 3 engine tries to limit your 'speed' to 320ups by the server variable <b>g_speed</b>. The engine doesn't allow you to accelerate past this 320ups in any one direction. Indeed, if you are travelling through the air at 320ups (or greater) and you hold the Forwards key (while facing the direction of travel) nothing will happen, because the engine is enforcing this limit, whether you are running or flying. So you have to use strafe-jumping to accelerate past 320ups.


Before I go any further, I will remind you of the following definitions:


  * _**Vector** = A quantity which has both a size and direction.
  * **Velocity** = A vector consisting of speed and direction of speed.
  * **Acceleration** = The rate of change of velocity with time. Therefore it is also a vector. The acceleration accounts for both the change in speed, and change in direction of the velocity._



So let's look at what happens when you are strafe-jumping. In all techniques, you need to be in the air at all times to avoid the friction caused by surfaces. This is the reason why players repeatedly jump, and this repeated jumping is known as bunny-hopping.


The first example technique is of standard <b>single-beat</b> strafing.


There are two air 'modes' involved in single-beat strafing, one being a mirror image of the other. In both modes, the player is orientated at an angle to the direction of travel. We shall call this angle <b>theta</b>, <span class="var">θ</span>. At the same time, the player is holding <b>Forwards</b> and either <b>Left Strafe</b> for the left mode, or <b>Right Strafe</b> for the right mode. This results in the following situation.


<img src="{{ "/images/strafe_theory/mode-left.gif" | prepend: site.baseurl }}" alt="Left mode"> <img src="{{ "/images/strafe_theory/mode-right.gif" | prepend: site.baseurl }}" alt="Right mode">


The black arrow is the <b>velocity</b> vector, which illustrates the <b>direction in which the player is currently moving</b>.<br>
The blue arrow is the <b>orientation</b> of the player (the direction in which you are <b>looking</b>).<br>
The red arrow is the <b>acceleration</b> vector.<br>


To perform the technique, the player quickly switches between modes (called an 'air-change') and continues alternating in this fashion by air-changing at (usually) regular intervals, usually in time with a jump.


Now keeping this in mind, let's look at the other distinctive strafing technique: half-beat strafing. You will see that half-beat strafing is actually the same thing as single-beat strafing, it's just another way to perform it.


Like single-beat, there are two air modes in half-beat. For lack of better terms, I shall name them 'first mode' and 'second mode'.


<img src="{{ "/images/strafe_theory/mode-left.gif" | prepend: site.baseurl }}" alt="First mode"> <img src="{{ "/images/strafe_theory/mode-second.gif" | prepend: site.baseurl }}" alt="Second mode">


The first mode is exactly the same as the 'left mode' from single-beat. The second mode appears a little strange at first. In this mode, the player is again orientated to the left of the direction of travel, by a small angle, which I have named angle <b>lambda</b>, <span class="var">λ</span>. At the same time, the player is holding <b>Right Strafe only</b>.


Now if you look back at the single-beat diagrams and compare them with these half-beat diagrams, you can see that the acceleration vectors (red arrows) are exactly the same in both techniques! There is a red arrow to the left in both the strafing techniques followed by one to the right. Plus, these acceleration vectors are all at the same angle to the velocity (use your imagination if necessary).


Indeed, it's possible to strafe in a number of ways, by using any combination of keys-presses and orientations in order to apply acceleration vectors in the way illustrated above. They are all equivalent, although some are more difficult to humanly perform than others.


Now let's look at why this angle between our velocity and acceleration is significant. We already know that the engine won't allow a player to gain speed in the direction of travel using the movement keys. However, we CAN gain speed in another direction, as long as our velocity's component is less than 320ups in that specific direction.


<img src="{{ "/images/strafe_theory/delta.gif" | prepend: site.baseurl }}" alt="delta">

So when we try to apply an acceleration at some angle to the velocity, we will be allowed to accelerate provided that the component of our current velocity in the proposed direction is less than 320ups. Mathematically, this is as follows.


Let's call this angle between our velocity and acceleration angle <b>delta</b>, <span class="var">δ</span>.


The component of the velocity in the direction of the acceleration is given by


<img src="{{ "/images/strafe_theory/math-dotproduct.gif" | prepend: site.baseurl }}" alt="Velocity component">.


This can be thought of as the projection of the velocity onto the direction of <span class="vector">a</span>.


As described above, the condition for acceleration to be allowed is


<img src="{{ "/images/strafe_theory/math-condition.gif" | prepend: site.baseurl }}" alt="Acceleration condition">.


Note that when <img src="{{ "/images/strafe_theory/math-vcos.gif" | prepend: site.baseurl }}" alt="v cos delta"> is negative, this means you are applying an acceleration in a direction at more than 90º to the velocity, which means you are effectively opposing your current velocity, a braking effect<a href="https://wayback.archive.org/web/20141224180047/funender.com/quake/articles/strafing_theory.html#asterisk">*</a>.


The size of the acceleration applied, <span class="var">a</span>, is a constant, independent of the angle at which it is applied (in the allowed region). This is defined by the engine in the same way that jump velocity and friction (etc.) are defined. However there is one special case which is an exception to this rule. If your <img src="{{ "/images/strafe_theory/math-vcos.gif" | prepend: site.baseurl }}" alt="v cos delta"> is less than 320 but still very close to 320, applying the usual constant acceleration for a single frame would be enough to take you over 320ups in that direction. This is something the engine checks for, and if the situation exists, the applied acceleration will be only enough to take you up to the 320ups limit (so it would be <img src="{{ "/images/strafe_theory/math-exception.gif" | prepend: site.baseurl }}" alt="320-vcostheta">), and not beyond. However, the range of angle delta at which this situation exists is very small.


The acceleration causes a change in the total velocity, which is made up of a <b>change in speed</b> and a <b>change in direction</b> (remember that velocity is a vector). Although the amount of acceleration, <span class="var">a</span>, is a constant, the direction that it is applied in dictates how much of this acceleration goes into changing the speed, and how much goes into changing the direction. In strafe-jumping it is the <b>change in speed</b> that we are interested in <b>maximising</b>, in an increasing fashion.


The way to achieve this maximum increase in speed is by applying the acceleration so its direction is as close to that of the velocity as possible, i.e. so the angle delta is as small as possible. This will mean that more of the acceleration goes into increasing our speed, and less goes into affecting the direction. The limiting condition gives a minimum angle delta of


<img src="{{ "/images/strafe_theory/math-deltamin.gif" | prepend: site.baseurl }}" alt="Minimum angle delta">.


However in order to gain the full acceleration, <span class="var">a</span>, we need the angle to be a bit larger, so we avoid the special case mentioned above. Therefore the optimal angle is the smallest angle at which we receive the full acceleration, <span class="var">a</span>, which is given by


<img src="{{ "/images/strafe_theory/math-deltaopt.gif" | prepend: site.baseurl }}" alt="Optimal angle delta">.


As you can see, this optimal angle is dependent on the speed, <span class="var">v</span>. This means that the optimal angle increases as you speed up, tending towards 90º at infinity.


Now that we know how acceleration is applied, we can look at the effect it has on the velocity, and the new velocity that results.


<img src="{{ "/images/strafe_theory/vectors.gif" | prepend: site.baseurl }}" alt="Vector addition">


This acceleration, <span class="vector">a</span>, causes the current velocity, <span class="vector">v</span>, to change and produces a resultant velocity, <span class="vector">r</span>. We can use basic trigonometry to calculate the size and change in direction of the resultant velocity. The change in direction also illustrates why strafe-jumping can cause the player to 'drift' to one side if the strafing is not consistent.


The results of these calculations are here for the mathematically-gifted; however it's not necessary to understand the following. From now on I shall use the parameter <span class="var">s</span> to refer to the value of g_speed. First let's look at the resultant speed after one frame. Note that there are three regions where acceleration is applied differently and hence the resultant speed is given by the following piecewise function.


<img src="{{ "/images/strafe_theory/math-resultant.gif" | prepend: site.baseurl }}" alt="Resultant velocity">


The acceleration per frame, <span class="var">a</span>, is defined (for air) by the engine as


<img src="{{ "/images/strafe_theory/math-accel.gif" | prepend: site.baseurl }}" alt="a=sT">,


where <span class="var">T</span> is the frame-time in seconds. So for a constant 125fps framerate and standard g_speed of 320, then <img src="{{ "/images/strafe_theory/math-accel2.gif" | prepend: site.baseurl }}" alt="a=2.56">  ups per frame.


The best way to interpret this is graphically. Here is a graph of the speed gained in a single frame, as a function of the applied angle delta.


<img src="{{ "/images/strafe_theory/angle-delta.gif" | prepend: site.baseurl }}" alt="plot of angle delta">


Although this graph looks triangular, the two curves are not actually straight lines, but as you can see, for all intents and purposes they can be thought of as straight lines. Also note that the spacing between the minimum and optimal delta angles has been exaggerated in the graph, and so the increase in speed drops off very quickly as angle delta is decreased from its optimal value.


This makes it highly important to ensure that any applied angle delta is not too small. It's not so bad if the angle is a little larger than optimal, but if it is smaller than optimal, the consequences are much greater. Also remember that both the minimum and optimal angles are velocity-dependent, so they both move closer to 90º (and closer to each other) as the velocity increases. During normal strafe-jumping these two angles are separated by less than 0.3°, so you can imagine how steep that part of the graph above really looks. There is, therefore, a very fine line between an optimal and zero speed increase.


If we apply an optimal angle delta, we get a resultant velocity of


<img src="{{ "/images/strafe_theory/math-resultantopt.gif" | prepend: site.baseurl }}" alt="optimal resultant velocity">,


which we can use to graph the <b>maximum increase in speed per frame</b>:


<img src="{{ "/images/strafe_theory/velocityincr.gif" | prepend: site.baseurl }}" alt="max speed increase per frame">


This graph illustrates how you cannot increase your speed as much at high velocites as you can at lower velocities.



Now we shall look at the change in direction. Let angle phi-one, <span class="var">φ</span><sub>1</sub>, be the change in direction due to one frame of acceleration, at our optimal delta angle.


<img src="{{ "/images/strafe_theory/math-phi1.gif" | prepend: site.baseurl }}" alt="phi-one">


When strafing, the direction in which you are travelling is <b>always changing</b>. However, you cannot see the direction that your velocity is pointing in, and it's difficult to guess accurately just by looking, so it can be very difficult to apply the optimal angle delta unaided.


It is much more useful to know the angle that we must apply relative to the overall direction in which we are trying to move in, e.g. from the beginning of a strafe pad lane to the end. I shall refer to this direction as the 'direction of overall motion'.


To do this, we need to consider how much our velocity direction changes from one bunny-hop to the next. Let angle phi, <span class="var">φ</span>, be the total change in direction caused by strafing between air-changes.


<img src="{{ "/images/strafe_theory/zigzag.gif" | prepend: site.baseurl }}" alt="Zig-zag path">


If we analyse the change in direction per frame, we see that not only is it small, but it also doesn't change much over the period between air-changes, provided that the velocity is greater than about 600ups. This is useful to us, since 600ups is roughly the boundary between the end of a circle-jump, and the beginning of strafe-jumping. This is also the reason why the technique used for circle-jumps needs to be different to strafe-jumping.


We can use this to gain an expression for the change in direction between air-changes.


<img src="{{ "/images/strafe_theory/math-phia.gif" | prepend: site.baseurl }}" alt="phi">
<br>
Phi is the change in direction between air-changes. <span class="var">N</span> is the number of frames between air-changes.   <img src="{{ "/images/strafe_theory/math-phi1avg.gif" | prepend: site.baseurl }}" alt="phi-one average"> is the average change in direction per frame. [Using standard gravity (800) and 125fps, there are about 88 air-frames between bunny-hops.]


<img src="{{ "/images/strafe_theory/math-phib.gif" | prepend: site.baseurl }}" alt="phi">


Therefore we can take this change in direction into account, and specify the strafing angles <b>relative to the fixed direction of overall motion</b>, rather than giving them relative to the velocity direction, which is always changing.


First let's define angle alpha, <span class="var">α</span>, as the angle between the direction of overall motion and the optimal acceleration direction, at an air-change.


<img src="{{ "/images/strafe_theory/math-alpha.gif" | prepend: site.baseurl }}" alt="alpha">


Then we can go on to apply this to the different strafing techniques.


The first strafing angle I shall call angle <b>epsilon</b>, <span class="var">ε</span>. This is the angle that is used to implement an optimal angle theta, used in <b>single-beat</b> strafing and for the 'first mode' of half-beat. (Here the orange arrow is the direction of overall motion)


<img src="{{ "/images/strafe_theory/sb2.gif" | prepend: site.baseurl }}" alt="Left/First mode">


Angle epsilon is thus,


<img src="{{ "/images/strafe_theory/math-epsilon.gif" | prepend: site.baseurl }}" alt="epsilon">.


If we look at the graph of angle epsilon, we can see that the angle <b>increases</b> as we speed up.


<img src="{{ "/images/strafe_theory/angle-epsilon.gif" | prepend: site.baseurl }}" alt="plot of angle epsilon">



The other strafing angle I shall call angle <b>kappa</b>, <span class="var">κ</span>. This is the angle that is used to implement an optimal angle lambda, used for the 'second mode' of <b>half-beat</b>.


<img src="{{ "/images/strafe_theory/hb2.gif" | prepend: site.baseurl }}" alt="Second mode">


Angle kappa is thus,


<img src="{{ "/images/strafe_theory/math-kappa.gif" | prepend: site.baseurl }}" alt="kappa">


Looking at the graph of angle kappa, we can see that angle kappa becomes smaller as we speed up, and is smaller in general than angle epsilon.


<img src="{{ "/images/strafe_theory/angle-kappa.gif" | prepend: site.baseurl }}" alt="plot of angle kappa">




In short, the strafing angle for single-beat (epsilon) begins at about 22º and increases. It's about 33º at 1000ups. The half-beat angle (kappa) begins at about 23º and decreases. It's about 12º at 1000ups.



<a id="asterisk">*</a> There is a small angle by which angle delta can exceed 90º and still result in a speed increase. The maximum angle delta is given by


<img src="{{ "/images/strafe_theory/math-deltamax.gif" | prepend: site.baseurl }}" alt="maximum delta">.


This is the boundary between a speed increase and a speed decrease.
