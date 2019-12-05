+++
title = "Robot Localization"
author = ["Jethro Kuan"]
lastmod = 2019-12-05T14:08:12+08:00
draft = false
math = true
+++

Mobile robot localization is the problem of estimating the robots
coordinates in an external reference frame from sensor data, relative
to a given map of the environment.

The robot's momentary estimate (belief) is represented by a
probability density function over the space of all locations. A
uniform distribution as prior represents maximal uncertainty.

Localization can be seen as a problem of coordinate transformation.
Maps are described in the global coordinate system, independent of the
robot's pose. Localization is the process of establishing
correspondence between the map coordinate system and the robot's local
coordinate system.

Pose often cannot be sensed directly, and has to be inferred from
data. A single sensor measurement is usually insufficient, and
integrating data over time is required to determine the pose.


## Taxonomy {#taxonomy}


### Local vs Global localization {#local-vs-global-localization}

position tracking
: initial robot pose is known. Localization is
    achieved by accommodating the noise in robot motion. The problem is
    local, since the uncertainty is local and confined to the region
    near the robot's true pose.

global localization
: initial robot pose is unknown. Robot is
    initially placed somewhere in its environment, but it lacks
    knowledge of where it is.

kidnapped robot problem
: During operation, the robot can get
    kidnapped and teleported to some other location. The robot might
    believe it knows where it is, while it does not.


### Static vs Dynamic Environment {#static-vs-dynamic-environment}

static environment
: only variable quantity is the robot's pose.
    Other objects remain at the same location forever.

dynamic environment
: there exists objects whose location or
    configuration may change over time.


### Passive vs Active Approaches {#passive-vs-active-approaches}

passive localization
: localization module only observes the robot
    operating: the robot is controlled via other means.

active localization
: control the robot as to minimize the
    localization error or costs arising from moving a poorly localized
    robot into a hazardous place.


## Markov Localization {#markov-localization}

A direct extension of the [§bayes\_filter]({{< relref "bayes_filter" >}}), but using the map \\(m\\) of the
environment.

\begin{algorithm}
  \caption{Markov Localization}
  \label{markov\_localization}
  \begin{algorithmic}[1]
    \Procedure{Markov Localization}{$\text{bel}(x\_{t-1}), u\_t, z\_t, m$}
    \ForAll{$x\_t$}
    \State $\overline{\text{bel}}(t) = \int p(x\_t | u\_t, x\_{t-1}, m)
    \text{bel}(x\_{t-1}) dx$
    \State $\text{bel}(t) = \eta p(z\_t | x\_t, m)\overline{\text{bel}}(t) (x\_t)$
    \EndFor
    \State \Return $bel(x\_t)$
    \EndProcedure
  \end{algorithmic}
\end{algorithm}

The initial belief reflects initial knowledge of the robot pose, and
can be instantiated differently:

If the initial pose is known, \\(\mathrm{bel}(x\_0)\\) is a point-mass
distribution such that:

\begin{equation}
  \operatorname{bel}\left(x\_{0}\right)=\left\\{\begin{array}{ll}{1} & {\text { if } x\_{0}=\bar{x}\_{0}} \\ {0} & {\text { otherwise }}\end{array}\right.
\end{equation}

However, point-mass distributions are discrete and do not have a
density, so in most scenarios, a narrow Gaussian centered around
\\(\overline{x}\_0\\) is used instead.

If the initial pose is unknown, \\(\mathrm{bel}(x\_0)\\) is initialized
with a uniform distribution over the space of all legal poses in the map.


## EKF localization {#ekf-localization}

EKF localization assumes that the map is represented by a collection
of features. At any point in time \\(t\\), the robot gets to observe a
vector of ranges and bearings to nearby features:
\\(z\_{t}=\left\\{z\_{t}^{1}, z\_{t}^{2}, \ldots\right\\}\\).

The easiest localization algorithm assumes all features are uniquely
identifiable, although this assumption can be corrected for. This
problem is called the unknown correspondence problem. One solution is
the maximum likelihood correspondence, where one determines the most
likely value of the correspondence and uses that value.

Maximum likelihood correspondence is brittle when there are many
equally likely hypotheses for the correspondence variable. This is
skirted around by choosing landmarks that are sufficiently distinct or
far apart, and making sure that the robot's uncertainty is small.


### Practical Considerations {#practical-considerations}

efficient search
: it is computationally expensive to loop through
    all landmarks \\(k\\) in the map. There are often simple tests to
    identify plausible candidate landmarks.

mutual exclusion
: a key assumption is the independence of feature
    noise, which enabled us to process individual features sequentially,
    avoiding an exponential search. This resulted in being able to
    assign the same landmark in the map with multiple values. However,
    this is invalid by default in some sensors, such as cameras. Knowing
    this mutual exclusion can significantly reduce the search space.

outliers
: Outliers are not addressed in the algorithm.

information
: EKF localization uses a subset of all available
    information (processed features) to localize. In addition EKF is
    unable to process negative information (lack of a feature).