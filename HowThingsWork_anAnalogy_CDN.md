# CDN -- an analogy

## Background
You are top-tier connoisseur with the most exquisite palate, and your favorate go-to comfort food is _Shiba Wuff_, and international chain restaurant lol.

## Scenario
You are sitting in your hotel room after a long day of restaurant hopping with smallest bites possible.

You are tired, your tongue is tired, you want food, good food, good comfort food, good _Shiba Wuff_ food ! 

Oh it absolutely has to be there when you want it, now where is the closest _Shiba Wuff_? 

A people person as you are, you decide to call your personal butler, `Butler`, directly.

### How it works, a conversation

#### Convo1 (`client` <-> `DNS resolver`)
- You: hello, my favorate butler, howdy! i'm hangry, find me addr of the closest _Shiba Wuff_ restaurant pronto please, dont wanna walk! thanks! 
Oh I'm in Hamburg Hauptbahnhof atm.
- `Butler`: no problemo boss, hold the line!

#### Convo2 (`DNS resolver` <-> `Authoritative Name Server for website`, resolves to `cdn`)
`Butler` looks up his little notebook, blanks... so he calls up his contact buddy, `C1`:
- `Bulter`: hey there, long time! Do you know where is _Shiba Wuff_ in this dreaful town, I mean, Hamburg Hauptbahnhof?
- `C1`: a sec! Well, aren't you lucky! We are not one of those snobbish exclusive restaurants with only one branch in the world,
that would so sad right, every time you want to dine with us you have to take all the trouble to travel lol!  **_(in the case WITHOUT cdn)_**.
We decided that we shall have branches all over the world, we have to be wherever you crave our food **_(in the case WITH cdn)_**! Here, this is the no. of the **contact person from their headquater**, `C2`.
Wuff be with you.
- `Butler`: Wuff right back at you.

#### Convo3 (`DNS resolver` <-> `Authoritative Name Server for CDN`, resolves to `lb`, short for load balancer)
`Butler` a thorough butler as he is, wastes no time calling `C2` up, he's determined to get to the bottom of it!
- `Butler`: here there! I would like the addr of your nearest branch to Hamburg Hauptbahnhof?
- `C2`: Oh let me get your our director in branches, `C3`, **a know-it-all of all branches we have**!

#### Convo3 (`DNS resolver` <-> `CDN load balancer`, resolves to the finall addr)
`Butler` could almost see the light at the end of the tunnel!
- `Butler`: Yo `C3`! where's your branch with minimum walk possible from Hauptbahnhof in your city please, i'm heading there now!
- `C3`: Oh we have several branches in Hamburg, the one currently with vacancies and closest to you is at this addr, **710, Shiba Allee, Hamburg**!
- `Butler`: wuf!
- `C3`: wuf!

#### Convo1.cont (`DNS resolver` <-> `client`, returns the response)
`Butler` finally gets back to the line you have been holding in hu(a)nger:
- `Butler`: **710, Shiba Allee, Hamburg**, bon appetite yo.
- You: oh thanks!

Addr in hand, and off you go! Bon Appetito!









