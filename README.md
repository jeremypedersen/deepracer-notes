# DeepRacer Journal/Model Bank
**Latest Update: 2024-09-02**

**Disclaimer:** At the time of writing, I am employed by Amazon as an SSA (Specialist Solutions Architect) in Beijing, China. *However, the notes, opinions, and thoughts on DeepRacer modeling shared here are my own, not those of my employer.* In cases where I borrow ideas or methods from other people, I try to make that clear with appropriate references.

The idea to keep a DeepRacer journal comes from [Scott Pletcher's awesome repo](https://github.com/scottpletcher/deepracer
), where he documents some of the reward functions he has tried and the logic behind them.

## Starting Out

I started by taking the (free) [AWS DeepRacer: Driven By Reinforcement Learning](https://explore.skillbuilder.aws/learn/course/internal/view/elearning/87/aws-deepracer-driven-by-reinforcement-learning) course on AWS Skillbuilder. 

The course runs through the basics of setting up the physical DeepRacer car, using the DeepRacer console, and training and evaluating a model. It also explains what the model's hyperparameters are, and what parameters you can use in the reward function to reward (or punish) your car for taking certain actions.

During the course, I used a few of the example reward functions provided [in the DeepRacer documentation](https://docs.aws.amazon.com/deepracer/latest/developerguide/deepracer-reward-function-input.html) and trained the car on the **Jennens Family Speedway** track, typically for 1 hour at a time.

### Observations

After playing with some of the provided functions, I learned that:
- Complicated reward functions aren't always better (in fact, making a lot of assumptions about what the car *"should"* be doing seems to be a bad thing)
- Some reward functions can be trained more quickly than others (in general, following the centerline leads to fast convergence)

With these observations in mind, I opted to start with simpler reward functions, only adding new pieces as necessary to coax the model towards desired behaviors it had not acquired on its own. 

## 1. The First Few Models

| Model | Purpose |
|-------|---------|
| [Follow the line](models/follow_the_line.md) | Just follow the centerline |
| [Don't wiggle](models/dont_wiggle.md) | Follow the centerline, but penalize the car for steering angles > 15 degrees |
| [Stay on track](models/stay_on_track.md) | Reward the car based on its ability to stay on the track and reasonably close to the centerline |

All three of these models were trained on the Jennens Family Speedway track: ***none* of them achieved a time under 1 minute, and *only* the *Don't Wiggle* model managed to make it around the track without any resets.**

## 2. Trying For More Speed

The obvious next step was to try and get the car moving faster. To do this, I cloned my three models again, but with a **modified reward function that penalized low speeds**. Specifically, I penalized the car for speeds below 1 m/s, by reducing the reward by a factor of `0.8` (for the *Don't wiggle* model, the reward was reduced even further if the car's steering angle was > 15 degrees). I also updated the maximum allowed speed in the *Action space* settings, from 1 m/s to 2 m/s. 

| Model | Purpose |
|-------|---------|
| [Follow the line, fast](models/follow_the_line_fast.md) | Same as *Follow the line*, but with a low speed penalty |
| [Don't wiggle, fast](models/dont_wiggle_fast.md) | Same as before (over-steering penalty), but with a low speed penalty added as well |
| [Stay on track, fast](models/stay_on_track_fast.md) | Same as before, but with a low speed penalty |

The results were good: the car *did* speed up, and most models could complete the track in under a minute (under 40 seconds, in some cases). 

### Still More Speed? 

Rather than punishing low speeds, I cloned my models again and tried directly adding a scaling factor to the reward, which scaled up as the car traveled faster. I also raised the car's maximum speed to 3 m/s. 

The scaling factor looked something like this:

```python
# Give a bonus for high speeds
reward += speed / 3.0
```

I was expecting good results, so I was surprised when the models performed badly. Instead of speeding around the track, the models were flying off it! Perhaps the reward for speed was overwhelming the other rewards for staying on the track and/or staying near the center.

I ended up throwing these models away to go back to the drawing board.

