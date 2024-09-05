---
title: Overall Functions
layout: default
has_toc: true
has_children: false
nav_order: 2
---



# Get PKL Standings 


## `get_pkl_standings(season=None, qualified=False, team_id=None`


Retrieve the Pro Kabaddi League (PKL) standings for a specified season.
 
#### Parameters:

- `season` : int, optional
    The season number for which to retrieve standings. Defaults to 10 if not specified.
- `qualified` : bool, optional
    If True, returns an additional DataFrame with only qualified teams. Defaults to False.
- `team_id` : int, optional
    If specified, returns standings for only this team. Defaults to None (all teams).

#### Returns:

pandas.DataFrame or tuple of pandas.DataFrame
    If qualified is False:
        Returns a DataFrame containing standings for all teams.
    If qualified is True:
        Returns a tuple of two DataFrames:
        (qualified_teams_standings, all_teams_standings)


The DataFrame(s) include the following columns:
- **Group**: The group name (if applicable)
- **Season**: The season number
- **Team_Id**: Unique identifier for the team
- **Team_Name**: Name of the team
- **League_position**: Current position in the league
- **Matches_played**: Number of matches played
- **Wins**, **Lost**, **Tied**, **Draws**: Match outcomes
- **No Result**: Number of matches with no result
- **League_points**: Total points in the league
- **Score_diff**: Score difference
- **Qualified**: Boolean indicating if the team qualified



#### Note:

If the standings data for the specified season is empty, an empty DataFrame is returned.


### Example Usage

```python
    from kabaddiPy import KabaddiDataAPI as kabaddi

    api = kabaddi()

    qualified_df, all_standings_df = api.get_pkl_standings(season=9, qualified=True)
    print("Qualified teams:")
    print(qualified_df)
    print("\nAll standings:")
    print(all_standings_df)
```

Output:
```
Qualified teams:
  Group  Season  Team_Id             Team_Name League_position Matches_played Wins Lost Tied Draws No Result League_points Score_diff  Qualified
0  Main       9        3  Jaipur Pink Panthers               1             22   15    6    1     0         0            82        174       True
1  Main       9        7         Puneri Paltan               2             22   14    6    2     0         0            80         66       True
2  Main       9        1       Bengaluru Bulls               3             22   13    8    1     0         0            74         39       True
3  Main       9       30            UP Yoddhas               4             22   12    8    2     0         0            71         42       True
4  Main       9       29       Tamil Thalaivas               5             22   10    8    4     0         0            66          5       True
5  Main       9        2     Dabang Delhi K.C.               6             22   10   10    2     0         0            63         17       True

All standings:
   Group  Season  Team_Id             Team_Name League_position Matches_played Wins Lost Tied Draws No Result League_points Score_diff  Qualified
0   Main       9        3  Jaipur Pink Panthers               1             22   15    6    1     0         0            82        174       True
1   Main       9        7         Puneri Paltan               2             22   14    6    2     0         0            80         66       True
2   Main       9        1       Bengaluru Bulls               3             22   13    8    1     0         0            74         39       True
3   Main       9       30            UP Yoddhas               4             22   12    8    2     0         0            71         42       True
4   Main       9       29       Tamil Thalaivas               5             22   10    8    4     0         0            66          5       True
5   Main       9        2     Dabang Delhi K.C.               6             22   10   10    2     0         0            63         17       True
6   Main       9       28      Haryana Steelers               7             22   10   10    2     0         0            61         16      False
7   Main       9       31        Gujarat Giants               8             22    9   11    2     0         0            59        -16      False
8   Main       9        5               U Mumba               9             22   10   12    0     0         0            56        -28      False
9   Main       9        6         Patna Pirates              10             22    8   11    3     0         0            54        -58      False
10  Main       9        4       Bengal Warriorz              11             22    8   11    3     0         0            53        -12      False
11  Main       9        8         Telugu Titans              12             22    2   20    0     0         0            15       -245      False

```

    def get_season_matches(self, season="all"):
        """
        Retrieve match data for a specific season or all seasons.

        This function loads match data from JSON files and returns it as a pandas DataFrame.

        Parameters:
        -----------
        season : str or int, optional
            The season number for which to retrieve match data. 
            Use "all" to retrieve data for all seasons (default).
            If a specific season is desired, provide the season number as a string or integer.

        Returns:
        --------
        pandas.DataFrame
            A DataFrame containing match details with the following columns:
            - Season: The season number
            - Match_ID: Unique identifier for the match
            - Match_Name: Name of the match event
            - League_Stage: Stage of the league (e.g., group stage, playoffs)
            - Year: Year of the match
            - Venue: Location where the match was played
            - Match_Outcome: Outcome of the match
            - Start_Date: Start date and time of the match
            - End_Date: End date and time of the match
            - Result: Result code of the match
            - Winning Margin: Margin of victory
            - team_score_1: Score of the first team
            - team_score_2: Score of the second team
            - team_name_1: Name of the first team
            - team_id_1: ID of the first team
            - team_name_2: Name of the second team
            - team_id_2: ID of the second team

        Notes:
        ------
        - The function reads data from JSON files located in the './Matches-Overview/' directory.
        - For "all" seasons, it sorts the files based on the season number extracted from the filename.
        - Each row in the returned DataFrame represents a single match.
        """
        
        matches_list = []

        # Determine the file(s) to load based on the season input
        if season == "all":
            files = glob.glob('./Matches-Overview/S*_PKL_MatchData.json')
            # Sort the files based on the season number
            files = sorted(files, key=lambda x: int(re.search(r'S(\d+)', x).group(1)))
            print(files)

        else:
            files = [f'./Matches-Overview/S{season}_PKL_MatchData.json']

        for file in files:
            
            with open(file) as f:
                data = json.load(f)


            for match in data['matches']:

                team_name, team_id, team_score = [], [], []

                for p in match['participants']:
                    # print(p['name'])
                    team_name.append(p['name'])
                    team_id.append(p['id'])
                    team_score.append(p['value'])


                match_details = {
                    "Season": match['tour_name'].split(",")[0].split(" ")[-1],
                    'Match_ID': match['game_id'],
                    "Match_Name": match['event_name'],
                    "League_Stage": match['event_stage'],
                    "Year": match['tour_name'].split(",")[1].strip(),
                    "Venue": match['venue_name'].lower().title().strip(),
                    'Match_Outcome': match['event_sub_status'],
                    "Start_Date": match['start_date'],
                    "End_Date": match['end_date'],
                    "Result": match['result_code'],
                    "Winning Margin": match['winning_margin'],
                    'team_score_1': team_score[0],
                    'team_score_2': team_score[1],
                    'team_name_1': team_name[0],
                    'team_id_1': team_id[0],
                    'team_name_2': team_name[1],
                    'team_id_2': team_id[1],
                }

                

                matches_list.append(match_details)

        # Convert the list of dictionaries into a DataFrame
        df = pd.DataFrame(matches_list)

        # df.to_csv("matches_data.csv")

        # Display the DataFrame
        return df
