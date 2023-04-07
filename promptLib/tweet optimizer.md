# Tweet optimizer prompt

## Contents:

- [Ranking algo docs](#Ranking algo docs)
- [Thread prompt](#Thread prompt)
	- [System](#System)
	- [Prompt](#Prompt)
	- [Iteration prompt](#Iteration prompt)
- [Tweet prompt](#Tweet prompt)
	- [System](#System)
	- [Prompt](#Prompt)
	- [Iteration prompt](#Iteration prompt)

---

# Ranking algo docs


```
Twitter's Heavy Ranker recomendation algorithm
Overview

The heavy ranker is a machine learning model used to rank tweets for the "For You" timeline which have passed through the candidate retrieval stage. It is one of the final stages of the funnel, succeeded primarily by a set of filtering heuristics.

The model receives features describing a Tweet and the user that the Tweet is being recommended to. The model architecture is a parallel MaskNet which outputs a set of numbers between 0 and 1, with each output representing the probability that the user will engage with the tweet in a particular way. The predicted engagement types are explained below:

scored_tweets_model_weight_fav: The probability the user will favorite the Tweet.
scored_tweets_model_weight_retweet: The probability the user will Retweet the Tweet.
scored_tweets_model_weight_reply: The probability the user replies to the Tweet.
scored_tweets_model_weight_good_profile_click: The probability the user opens the Tweet author profile and Likes or replies to a Tweet.
scored_tweets_model_weight_video_playback50: The probability (for a video Tweet) that the user will watch at least half of the video.
scored_tweets_model_weight_reply_engaged_by_author: The probability the user replies to the Tweet and this reply is engaged by the Tweet author.
scored_tweets_model_weight_good_click: The probability the user will click into the conversation of this Tweet and reply or Like a Tweet.
scored_tweets_model_weight_good_click_v2: The probability the user will click into the conversation of this Tweet and stay there for at least 2 minutes.
scored_tweets_model_weight_negative_feedback_v2: The probability the user will react negatively (requesting "show less often" on the Tweet or author, block or mute the Tweet author).
scored_tweets_model_weight_report: The probability the user will click Report Tweet.

The outputs of the model are combined into a final model score by doing a weighted sum across the predicted engagement probabilities. The weight of each engagement probability comes from a configuration file, read by the serving stack here:

package com.twitter.home_mixer.product.scored_tweets.param

import com.twitter.conversions.DurationOps._
import com.twitter.home_mixer.param.decider.DeciderKey
import com.twitter.timelines.configapi.DurationConversion
import com.twitter.timelines.configapi.FSBoundedParam
import com.twitter.timelines.configapi.FSParam
import com.twitter.timelines.configapi.HasDurationConversion
import com.twitter.timelines.configapi.decider.BooleanDeciderParam
import com.twitter.util.Duration

object ScoredTweetsParam {
  val SupportedClientFSName = "scored_tweets_supported_client"

  object CrMixerSource {
    object EnableCandidatePipelineParam
        extends BooleanDeciderParam(DeciderKey.EnableScoredTweetsCrMixerCandidatePipeline)
  }

  object FrsTweetSource {
    object EnableCandidatePipelineParam
        extends BooleanDeciderParam(DeciderKey.EnableScoredTweetsFrsCandidatePipeline)
  }

  object InNetworkSource {
    object EnableCandidatePipelineParam
        extends BooleanDeciderParam(DeciderKey.EnableScoredTweetsInNetworkCandidatePipeline)
  }

  object QualityFactor {
    object MaxTweetsToScoreParam
        extends FSBoundedParam[Int](
          name = "scored_tweets_quality_factor_max_tweets_to_score",
          default = 1100,
          min = 0,
          max = 10000
        )

    object CrMixerMaxTweetsToScoreParam
        extends FSBoundedParam[Int](
          name = "scored_tweets_quality_factor_cr_mixer_max_tweets_to_score",
          default = 500,
          min = 0,
          max = 10000
        )
  }
  object ServerMaxResultsParam
      extends FSBoundedParam[Int](
        name = "scored_tweets_server_max_results",
        default = 120,
        min = 1,
        max = 500
      )
  object UtegSource {
    object EnableCandidatePipelineParam
        extends BooleanDeciderParam(DeciderKey.EnableScoredTweetsUtegCandidatePipeline)
  }

  object CachedScoredTweets {
    object TTLParam
        extends FSBoundedParam[Duration](
          name = "scored_tweets_cached_scored_tweets_ttl_minutes",
          default = 3.minutes,
          min = 0.minute,
          max = 60.minutes
        )
        with HasDurationConversion {
      override val durationConversion: DurationConversion = DurationConversion.FromMinutes
    }

    object MinCachedTweetsParam
        extends FSBoundedParam[Int](
          name = "scored_tweets_cached_scored_tweets_min_cached_tweets",
          default = 30,
          min = 0,
          max = 1000
        )
  }

  object Scoring {
    object HomeModelParam
        extends FSParam[String](name = "scored_tweets_home_model", default = "Home")

    object ModelWeights {

      object FavParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_fav",
            default = 1.0,
            min = 0.0,
            max = 100.0
          )

      object RetweetParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_retweet",
            default = 1.0,
            min = 0.0,
            max = 100.0
          )

      object ReplyParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_reply",
            default = 1.0,
            min = 0.0,
            max = 100.0
          )

      object GoodProfileClickParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_good_profile_click",
            default = 1.0,
            min = 0.0,
            max = 1000000.0
          )

      object VideoPlayback50Param
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_video_playback50",
            default = 1.0,
            min = 0.0,
            max = 100.0
          )

      object ReplyEngagedByAuthorParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_reply_engaged_by_author",
            default = 1.0,
            min = 0.0,
            max = 200.0
          )

      object GoodClickParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_good_click",
            default = 1.0,
            min = 0.0,
            max = 1000000.0
          )

      object GoodClickV2Param
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_good_click_v2",
            default = 1.0,
            min = 0.0,
            max = 1000000.0
          )

      object NegativeFeedbackV2Param
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_negative_feedback_v2",
            default = 1.0,
            min = -1000.0,
            max = 0.0
          )

      object ReportParam
          extends FSBoundedParam[Double](
            name = "scored_tweets_model_weight_report",
            default = 1.0,
            min = -20000.0,
            max = 0.0
          )
    }
  }

  object EnableSimClustersSimilarityFeatureHydrationDeciderParam
      extends BooleanDeciderParam(decider = DeciderKey.EnableSimClustersSimilarityFeatureHydration)

  object CompetitorSetParam
      extends FSParam[Set[Long]](name = "scored_tweets_competitor_list", default = Set.empty)

  object CompetitorURLSeqParam
      extends FSParam[Seq[String]](name = "scored_tweets_competitor_url_list", default = Seq.empty)
}

The exact weights in the file can be adjusted at any time, but the current weighting of probabilities (April 5, 2023) is as follows:

scored_tweets_model_weight_fav: 0.5
scored_tweets_model_weight_retweet: 1.0
scored_tweets_model_weight_reply: 13.5
scored_tweets_model_weight_good_profile_click: 12.0
scored_tweets_model_weight_video_playback50: 0.005
scored_tweets_model_weight_reply_engaged_by_author: 75.0
scored_tweets_model_weight_good_click: 11.0
scored_tweets_model_weight_good_click_v2: 10.0
scored_tweets_model_weight_negative_feedback_v2: -74.0
scored_tweets_model_weight_report: -369.0

Essentially, the formula is:

score = sum_i { (weight of engagement i) * (probability of engagement i) }

Since each engagement has a different average probability, the weights were originally set so that, on average, each weighted engagement probability contributes a near-equal amount to the score. Since then, we have periodically adjusted the weights to optimize for platform metrics.
```


---


# Thread prompt

###### System

```
you are the ultimate ai tweeter thread optimizer with great writing capabilities (style of ray breadbary, ernst heminguei). your job is to improve and optimize tweets given to you based on the provided twitter recomendations algorithm documentation, code and your own knowlege and skills.
```

###### Prompt

```
you are provided with a text, representing a twitter thread following this structure:

--- 1
TWEET 1
--- 2
TWEET 2
--- 3
etc

using your knowlege and tweeter recomendation algorithm provided to you, improve and optimize tweeter thread given to you to reach the best possible exposure and engagement. please don't change the writing style and retain tone of voice. feel free to use emojis sparingly. let's think this through step by step. begin by providing a detailed list with suggestions on how to improve each tweet in the thread.

thread to improve:
```

###### Iteration prompt

```
improve and optimize tweeter thread given to you to reach the best possible exposure and engagement. please don't change the writing style and retain tone of voice. feel free to use emojis sparingly.
```

---


# Tweet prompt

###### System

```
you are the ultimate ai tweet optimizer with great writing capabilities (style of ray breadbary, ernst heminguei). your job is to improve and optimize tweets given to you based on the provided twitter recomendations algorithm documentation, code and your own knowlege and skills.
```

###### Prompt

```
you are provided with a tweet.

using your knowlege and tweeter recomendation algorithm provided to you, improve and optimize tweet given to you to reach the best possible exposure and engagement. don't change the writing style and retain tone of voice. use emojis sparingly. let's think this through step by step. begin by providing a detailed list with suggestions on how to improve the tweet.

tweet to improve:
```

###### Iteration prompt

```
improve and optimize the tweet given to you to reach the best possible exposure and engagement. don't change the writing style and retain tone of voice. use emojis sparingly.
```
