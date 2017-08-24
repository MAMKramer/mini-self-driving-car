Saturday, 19th August 2017
------------------------

## Data

As of today, we have two datasets that are not small.

- turn_right - 7831 images on right turn only
- first_big_dataset - 15353 images on full track

We are driving on white.

should make git repo on these 2 to avoid data loss.


## Model
We are using variation of Nvidia-proposed architecture.

    Conv2D(24, (5, 5), strides=(2, 2), padding='valid')
    ELU()

    Conv2D(36, (5, 5), strides=(2, 2), padding='valid')
    ELU()

    Conv2D(48, (5, 5), strides=(2, 2), padding='valid')
    ELU()

    Conv2D(64, (3, 3), strides=(1, 1), padding='valid')
    ELU()

    Conv2D(64, (3, 3), strides=(1, 1), padding='same')
    ELU()

    Conv2D(64, (3, 3), strides=(1, 1), padding='same')
    model.add(ELU())

    Conv2D(64, (3, 3), strides=(1, 1), padding='same')
    model.add(ELU())

    Flatten()

    Dense(100)
    ELU()

    Dense(50)
    ELU()

    Dense(10)
    ELU()
    Dropout(0.5)

    Dense(3, activation='softmax')


Task: classification
loss: categorical cross entropy
optimizer: Adam
Infra/lib: keras



## Ideas, work in progress

Prepared data viewer. If needed, can try to minimize conflicting inputs
by hand.

Need to add richer model save name handling to minimize risk of
overwriting good results with new runs.



## Experiments

Right turn only, 100 epochs,
We are using regularization: random flip, random translation up to 10 pix.

Val acc stabilies at around 85% which is record so far.
Can even be theoretical maximum (when we gather data, we sometimes give
different steering for same inputs.)

Killed optimization after 63rd epoch.

Max acc history
epoch acc
0 65
2 80
4 82
8 84
20 84
36 85


Output file called third.json

Checking OOS by hand...

Shows reasonable traces of working :)
Needs more through testing in real world, but batteries are down.

In particular spots on real right turn, turns wheels to the right.

Next thing to do: Test more, next perhaps work on dataset augmentation.



Sunday, 20th August 2017
------------------------

In the morning I have tested model 'three'.
Works mixed. Sometimes it has pretty good runs, sometimes it does not.
Overall, as a proof-of-concept it is rather encouraging, suggests
that with sufficiently meaningful data the framework will run.
I would say around one in 3 runs works OK.


As an experiment, I have turned to manually adjusting  collected data.
I hope to adjust for straight-bias in the data.
When you are driving the car dynamically, it feels natural to adjust
for momentum of turn and let the car coast a bit forward with straight wheels.

You are way less likely to do it when doing it by hand.

In the data adjustment, a person is shown the bottom crop and has to decide
 where to go, using keyboard input. Then next frame is shown.
Klaudynka was adjusting datapoints 6000 to the end. I was adjusting
points 0-6000. I suspect that human agent behaviour may be noisy and
fluctuate over time.

Distribution of data - right turn only

left           0.116362
right          0.551284
no_steering    0.332354

Indeed, the data has shifted a lot to the 'right'. Previously it was a lot less
likely. Maybe average out with the previous data? Kinda funny idea.

Data still contains a lot of mess - especially the back frames are pretty bad.
Shuold get rid of them.


First run of the experiment is very bad, basically does not work at all.
Retraining neural net...0

Second run seems bad. Running testing on max val_acc, 27 epochs @ 83% OOS acc,
completely wrong.
Teststing on 11 epcohs @ 80% OOS acc same bad.

Across second and first run very weird turn left at beginning of straight
behavior. Strong signal!
