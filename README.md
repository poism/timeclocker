# potime
A simple bash CLI task timer that helps keep you focused, on task, and on time.
Your time logs are saved to plaintext timeclock.el file for processing with hledger and can optionally be pushed to google calendar.

This tool is designed to work along with other plaintext cli productivity tools.


## Installation

Tested on:
- Linux (Ubuntu 22.04.3 LTS with i3wm)
- MacOS (Intel, MacOS 13.2 Ventura)


```
# First ensure ~/bin/ exists and add ~/bin/ to your PATH if necessary..

git clone https://github.com/poism/potime.git
ln -s $(pwd)/potime/potime ~/bin/potime

```

## Optional Dependencies

This tool can function without any of the following, but they certainly make it better.

*ffmpeg* or *pplay* - for linux sound playback

*hledger* - plaintext accounting and timekeeping
[hledger](https://hledger.org/) is used to display timelog output.
If you do not have hledger, you will be given the option to download it within this directory during usage.

*gcalcli* - plaintext google calendar
[gcalcli](https://github.com/insanum/gcalcli) is used to add items to google calendar.
Gcalcli requires Google API keys scoped to Google Calendar so that setup can be complicated and totally unnecessary.

If you do wish to use `gcalcli`, your events will be added to the default calendar of gcalcli or you can specify via env var.
For instance if you make a Google Calendar named "TimeClocker", simply add `export GCALENDARNAME="TimeClocker"` to your `~/.bash_rc` file.
All future shells in which you run this app will add events to your TimeClocker calendar.



## Usage

1. Run the script with how many minutes you wish to devote to whichever account, and optional task description.
2. It displays a progress bar timer and at completion displays a system notification and plays your music or a bell.
3. Press CTRL+c to stop the music and end the timer.
4. It will allow you to extend the timer if you wish to continue working on the same task.
5. You will be given a chance to provide additional comments or tags"
6. If you have [gcalcli](https://github.com/insanum/gcalcli) installed, you will be asked if you wish to add the event to your google calendar.
7. It writes to your `~/$USER.timeclock` file.


*Special Characters*
Avoid using special characters for descriptions and comments, they will be removed.
For accounts use colon `:` to define account hierarchies. Do not use spaces in account names.


```
potime --help
         USAGE:  potime [minutes] [account:optionalsubaccount] '[optional description ; comments, tag:tag1,tag2]'
      EXAMPLES:  potime 25 POISM:DEV potime app; added help messages, tags:dev,poism,timeclocker'
   ENVIRONMENT:  export TIMECLOCKFILE=~/yourusername.timeclock; export GCALENDARNAME="UNSPECIFIED"
CURRENT CONFIG:
                 TIMECLOCKFILE: /home/sangpo/sangpo.timeclock
                 TIMECLOCKSOUND: /datapool/projects/POISM/repos/potime/alarm.ogg
                 GCALENDARNAME: UNSPECIFIED
```

## Usage Examples


### One example with outputs:
```

potime 45 POISM:adm 'get frustrated using google calendar'
______________________________________________________________

FILE: /home/sangpo/sangpo.timeclock
TIME: 45 minutes, starting at 12:00:00 PM
TASK: POISM:adm  get frustrated using google calendar
[########################################] 100% (1:0)^C

FINISHED: POISM:adm duration was 45 minutes and 0 seconds.

Do you wish to extend the current task timer? Enter additional integer minutes to extend, or [0/n/nothing] to end timer.
--> n


______________________________________________________________

i 2024/01/21 12:00:00 POISM:adm  get frustrated using google calendar
o 2024/01/21 12:45:00
______________________________________________________________

Balance for this POISM:adm account:

               0.75h  POISM:adm
--------------------
               0.75h

```

### More examples output ommitted

```

potime 90 POISM:dev 'potime app ; make a bash app as an act of pocrastination, tag:dev,docs,potime'

potime 30 PERS:lunch 

potime 15 POISM:dev 'potime app ; wrote readme and upload to github, tag:dev,docs,potime'

```

## Analyzing your timeclock examples

```
$ hledger -f ~/sangpo.timeclock bal -W
Balance changes in 2024-01-15W03:

            || 2024-01-15W03 
============++===============
 PERS:lunch ||         0.50h 
 POISM:adm  ||         0.75h 
 POISM:dev  ||         1.75h 
------------++---------------
            ||         3.00h 

```

```
$ hledger -f ~/sangpo.timeclock print -W
2024-01-21 * get fustrated using google calendar
    (POISM:adm)           0.75h

2024-01-21 * potime app ; make a bash app as an act of pocrastination, tag:dev,docs,potime
    (POISM:dev)           1.50h

2024-01-21 * 14:31-15:01
    (PERS:lunch)           0.50h

2024-01-21 * potime app ; wrote readme and upload to github, tag:dev,docs,potime
    (POISM:dev)           0.25h

```

```
$ hledger -f ~/sangpo.timeclock bal -p 2024/1
               0.50h  PERS:lunch
               0.75h  POISM:adm
               1.75h  POISM:dev
--------------------
               3.00h

```

```
$ hledger -f ~/*.timeclock bal -p 2024/1 --depth 1
               0.50h  PERS
               2.50h  POISM
--------------------
               3.00h

```

```
$ hledger -f ~/*.timeclock bal -D POISM
Balance changes in 2024-01-21..2024-01-22:

           || 2024-01-21  2024-01-22 
===========++========================
 POISM:adm ||      0.77h           0 
 POISM:dev ||      1.77h       0.37h 
-----------++------------------------
           ||      2.54h       0.37h 
```

```
$ hledger -f ~/*.timeclock bal -D --depth 1
Balance changes in 2024-01-21..2024-01-22:

       || 2024-01-21  2024-01-22 
=======++========================
 PERS  ||      0.50h       1.41h 
 POISM ||      2.54h       0.37h 
-------++------------------------
       ||      3.04h       1.78h 
```

## Custom alarm and audio caveats

You can add an `alarm.mp3` file and it will automatically be used if possible.

On Linux, `paplay` is assumed to not support mp3 in which case we fall back to the `alarm.ogg`, but if you have`ffplay` (hopefully included with `ffmpeg`) then the mp3 will be used.

On MacOS `afplay` does not support `.ogg` so we use a default bell sound, played once and then repeated for however many minutes of overage. If an mp3 is found that will be played instead.



## Further reading

- [hledger time budgets](https://hledger.org/time-planning.html#how-to-set-up-a-time-budget)
- [hledger for processing timeclock files](https://hledger.org/1.32/hledger.html#timeclock)

## Author
Sangpo Dorje
