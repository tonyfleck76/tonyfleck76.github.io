---
layout: post
title: "Baseball Superstitions"
keywords: data, analysis, statistics, rust, polars, dataframes, baseball, Chicago Cubs
summary: "Analyzing whether my favorite team truly performs worse when I watch them."
---

As a lifelong baseball fan, I've always found myself loving sports superstition. As a Cubs fan, the superstitions run deep. Making sure to wear your hat for the game, rally caps when going into the bottom of the ninth down a few runs,
and not washing your jersy during playoffs were all common things in our house. In fact, my dad had a bucket hat he wore all the time that we roasted him for endlessly. He was the one laughing when we finally saw the Cubs win
the World Series in 2016. Now the bucket hat has been proudly retired and placed in a shadow box, and my father continues his search for a new good luck charm.

Regardless of goat curses, unwashed jersies, or inside-out hats, I've long had a suspicion that I've definitely witnessed more losses for my teams than I have wins. Some people may say that's due to Chicago sports teams' incessant
mediocrity, but I refuse to accept that. All this leads me to an experiment I'm running this season.

This year I'm tracking every Cubs game I manage to watch. My plan was to stack up the Cubs' win rate of only games I watch to their win rate throughout the season. Now obviously this is a really simple calculation and I could 
do it in about 2 minutes on the back of a napkin. However, where's the fun in that? Obviously I'm going to automate it. Introducing: `For the Home Team`.

I spun up a quick Rust project (my new obsession) and pulled in Polars for dataframes. Now I needed data. Turns out, the MLB stats API is indeed open and available, though documentation is... hard to come by.
Thankfully, the Firefox network tab had everything I need. I managed to figure out the request I needed to make to pull in the team schedule, limiting to only games that have been completed. After a painstaking process
modeling the API response and figuring out how to flatten it out for Polars, I had a win/loss record to work with.

As an aside, one thing that is driving me crazy about Polars is that building dataframes seems _tedious_. By default, Polars builds series not as rows, but as columns. When I have an array of game objects, I would normally expect to map through them once,
building each game as a row. But in Polars I need to map through my games once for each column. That means once to map the IDs to the ID column series, Fan Team Name to another column, whether they won to a third column, and so on. It seems like the functionality
exists in the Python library, but not in Rust. Thankfully, I'm dealing with a very small amount of data so this isn't an issue so far, other than requiring way more code. The only thing I've found to get around this would be to put the data in a flattened struct,
serialize it to JSON and then read it into Polars from the JSON string. This feels hacky to me, so I haven't done it yet. It's a little easier in terms of how much code I need to write, but my gut tells me it's not the way to go. We'll see if that holds up as I
get further in.

Back to win rates, I now had the details I need. MLB games have an ID, so as an intermediate step I'm manually saving those into a text file. Those are read in as my `watched_games`. Then I load all the games the Cubs have played so far,
and build the following dataframe.

| game_pk | date | fan_team_name | fan_team_score | fan_team_winner | opposing_team_name | opposing_team_score | opposing_team_winner | watched_game |
|-------|--------|---------|-------|--------|---------|--------|---------|
| 718777 | 2023-03-30T18:20:00Z | Chicago Cubs | 4 | true | Milwaukee Brewers | 0 | false | true |
| 718758 | 2023-04-01T18:20:00Z | Chicago Cubs | 1 | true | Milwaukee Brewers | 3 | false | true |
| 718738 | 2023-04-02T18:20:00Z | Chicago Cubs | 5 | true | Milwaukee Brewers | 9 | false | true |

This will automatically fill out as the season continues, every time I run the program. Then with just a little extra work, I can calculate the separate win rates. Here's what that block of code looks like:

```rust
pub fn create_win_record_dataframe(
    game_dataframe: &DataFrame,
    total_game_count: usize,
    watched_games_count: usize,
) -> DataFrame {
    let overall_win_predicate = col("fan_team_winner")
        .filter(col("fan_team_winner"))
        .count()
        .alias("overall_wins");
    let overall_loss_predicate = col("fan_team_winner")
        .filter(col("fan_team_winner").eq(false))
        .count()
        .alias("overall_losses");
    let watched_game_win_predicate = col("fan_team_winner")
        .filter(col("fan_team_winner").and(col("watched_game")))
        .count()
        .alias("watched_wins");
    let watched_game_loss_predicate = col("fan_team_winner")
        .filter(col("fan_team_winner").eq(false).and(col("watched_game")))
        .count()
        .alias("watched_losses");

    game_dataframe
        .clone()
        .lazy()
        .select([
            overall_win_predicate,
            overall_loss_predicate,
            watched_game_win_predicate,
            watched_game_loss_predicate,
        ])
        .with_column(
            (col("overall_wins").cast(DataType::Float32) / lit(total_game_count as f32))
                .alias("overall_win_rate"),
        )
        .with_column(
            (col("watched_wins").cast(DataType::Float32) / lit(watched_games_count as f32))
                .alias("watched_win_rate"),
        )
        .collect()
        .unwrap()
}
```

And we end up with the following output:

| overall_wins | overall_losses | watched_wins | watched_losses | overall_win_rate | watched_win_rate |
|-------|--------|---------|-------|--------|---------|
| 4 | 4 | 3 | 1 | 0.5 | 0.6 |

There we have it! As of 8 games into the season, I'm technically at 60% wins, which is actually better than I thought. Now the stats bug has bit me, and I have lots more I want to do with this project. In the next post, I'm going to tackle batting averages, and see
if there's any difference in how any players perform in the games I watch. Unfortunately, the Mariners just tied the game I have on my second monitor, so I have other things to attend to. Stay tuned for more as I keep building this out! If you'd like to follow the stats, you can do so [here]({% link projects/forthehometeam.md %}).