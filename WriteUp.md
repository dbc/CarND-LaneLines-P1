# Reflections on Lane Finding

This was my first real attempt at computer vision and using OpenCV, so I
learned a lot even though I realize this is fairly basic image processing.  
I expect that I could improve the tuning for Canny and Hough if I had more 
time to spend on tuning the parameters.

## First Iteration

Under the git branch "homework-rev1" there is a simple pipeline that does 
reasonably OK on the first two videos but the output from the challenge 
video is very scruffy.  In rev1, lines are simply split into left/right 
based on slope, with unreasonable slopes discarded.  The remaining lines 
are extraplated to the clipping limit and the endpoints are averaged.

This performs well on the static images.  On the white and yellow videos it 
does OK, but the distant end points are not very stable.  On the challenge
video the output is scruffy, with lane marker end-points crossing the 
center line reasonbly often.

## Second Iteration.

For my second iteration (as submitted for review) I made the following
changes:

- In my line sorting, I discard lines where the extrapolated far end
  point crosses the center line, and average the remaining lines.  On
  a few video frames, this results in all lines from either the left
  or right batch being discarded.

- I attempted to smooth the behavior of the lane marks by averaging the
  lane lines of the four most recent video frames.  In essense, a four
  tap FIR filter with coeficients of [0.25, 0.25, 0.25, 0.25]. This does 
  smooth the lane lines somewhat, but still isn't perfect.  (Note that
  an artifact of my implementation is that since I am not clearing my
  filter buffer in the appropriate place, the first 3 frames of a new
  video are still averaging in data from the previous clip.  This is a
  simple problem to solve, but since I'm so behind I'm moving on to the
  second lesson.)

## Reflections on Second Submission

Based on reviewer feedback, I made several adjustments to the Hough
transform parameters.  This significantly stablized the video
of the challenge output.

Based on this experience, I would want a better test bench for tuning
the image pipeline parameters. I want to spend some time thinking about
how I can add that to my toolkit.

### Possible Failure Modes

Lighting: All of the photos and videos are very brightly and evenly lit.
Shadows may create false positive recognition of lane lines.  Night 
lighting is likely to be very challenging.

Noisey data: Extra markings of lane-line-like artifacts would add false
positive lines to the averages.  Things like turning lane markers,
tire marks, or even smeared lane line paint might be recognized as parts
of lane lines.

Tight curves and hills: The software extraplolates straight lines, it
does not attempt to recognize and fit curves.  Hills will cause perspective
distortion in otherwise straight lines.

Split lanes: Coming up to a road junction where the road splits from one
lane to two will sensitize a number of issues.  Lane lines may spread 
outside the ROI window, and when the lane actually splits there is an
ambiguity about which subsequent lane to follow.

### Possible Improvements

My method of selecting, extrapolating, and averaging lines is very basic.  
Some thoughts that come to mind are:

- Use some simple statistical analysis to throw out more "outlier" lines 
  before including them in the end-point average.

- Instead of extrapolating every line to the clipping top/bottom limit and 
  taking the average, fitting the line endpoints to a line might yield a
  better result although it would be more computationally intensive.

- Dynamically adjust the clipping window by using the current identified 
  lane lines to compute the corners of the clipping polygon for the next 
  frame.
  
- Account for upcoming curves in the lane lines by processing the image
  in multiple (say two or three) regions of interest based on distance.

- Explore different options for smoothing data across frames.  My current
  method of averaging the previous three lane lines is very simplistic.

- Do more extensive testing with images made under a wide variety of
  lighting conditions in order to find patter CV parameters, or perhaps
  create different sets of CV parameters, for instance day/night settings.




