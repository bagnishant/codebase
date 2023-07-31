# codebase
These are a few notable projects that I have worked on.
Many git repositories I worked on are private (Georgia Tech class rules + protection of company information), so I can’t fully share them; I have collected a few code samples and uploaded them here.
Code samples include Spotifind, Dungeon Crawler, and a few zavvie projects I worked on (MovieGrub is excluded because of IP agreements with the client)

## Dungeon Crawler:
Developed an interactive 2D-game with a group in which a player can defeat monsters, collect items, and navigate rooms. Used JavaFX platform for game development. Applied object oriented programming principles to design the game with handles to test for functionality.
### Monster.java
contains Monster entity class with details such as movement, health, damage, etc. The Monster entity can move around, deal damage to the player, and takes damage/dies.
```
package dungeoncrawler.entity.monster;
import dungeoncrawler.entity.Player;
import javafx.animation.KeyFrame;
import javafx.animation.KeyValue;
import javafx.animation.Timeline;
//import javafx.animation.Animation;
//import javafx.geometry.Bounds;
//import javafx.scene.control.Alert;
import javafx.scene.image.Image;
import javafx.scene.paint.ImagePattern;
import javafx.scene.shape.Rectangle;
import javafx.scene.paint.Color;
import javafx.scene.layout.Pane;
import javafx.util.Duration;
//import java.security.Key;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.util.Random;
import java.util.Timer;

// In case where you wanna create your own monster,
// just extends this class and implement startMoving
public class Monster extends Rectangle {
    private int health;
    private int damage = 1;
    private boolean alive = true;
    private boolean itemDropAvailable;

    public Monster(int width, int height, int health, Color color) {
        super(width, height, color);
        this.health = health;
        this.itemDropAvailable = true;
    }

    /**
     * Continuous function for animating the movement of the monster class.
     *
     * @param pane the pane the monster is in
     */
    public void move(Pane pane) {
        if (this.alive && Player.isAlive()) {
            Random rand = new Random();
            int maxValue = (int) Math.abs(pane.getWidth() - this.getWidth());
            int endX = rand.nextInt(maxValue);
            int endY = rand.nextInt(maxValue);
            double duration = 2.5;
            int maxDistance = (int) Math.max(Math.abs(endX - this.getX()),
                    Math.abs(endY - this.getY()));
            if (maxDistance <= 150) {
                duration = .8;
            } else if (maxDistance <= 275) {
                duration = 1.5;
            }
            KeyValue x = new KeyValue(this.layoutXProperty(), endX);
            KeyValue y = new KeyValue(this.layoutYProperty(), endY);
            KeyFrame frame = new KeyFrame(Duration.seconds(rand.nextDouble() + duration), x, y);
            Timeline timeline = new Timeline(frame);
            timeline.setCycleCount(1);
            timeline.play();
            timeline.setOnFinished(e -> {
                move(pane);
            });
        }
    }

    public void takeDamage(int damageCount) {
        this.health -= damageCount;
        if (this.health <= 0) {
            Player.killMonster();
            this.setVisible(false);
            this.alive = false;
        } else {
            if (this instanceof GreenMonster) {
                try {
                    this.setFill(new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\greenMonster2.png"))));
                    this.damageAnimation(this, new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\greenMonster.png"))));
                } catch (FileNotFoundException exception) {
                    System.out.println("Green monster image not found " + exception);
                }
            } else if (this instanceof PinkMonster) {
                try {
                    this.setFill(new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\pinkMonster2.png"))));
                    this.damageAnimation(this, new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\pinkMonster.png"))));
                } catch (FileNotFoundException exception) {
                    System.out.println("Pink monster image not found " + exception);
                }
            } else if (this instanceof YellowMonster) {
                try {
                    this.setFill(new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\yellowMonster2.png"))));
                    this.damageAnimation(this, new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\yellowMonster.png"))));
                } catch (FileNotFoundException exception) {
                    System.out.println("Yellow monster image not found " + exception);
                }
            } else if (this instanceof DogeMonster) {
                try {
                    this.setFill(new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\doge2.png"))));
                    this.damageAnimation(this, new ImagePattern(new Image(new FileInputStream(
                            System.getProperty("user.dir") + "\\res\\doge.png"))));
                } catch (FileNotFoundException exception) {
                    System.out.println("Doge monster image not found " + exception);
                }
            }
        }
    }

    public void damageAnimation(Monster monster, ImagePattern img) {
        Timer t = new java.util.Timer();
        t.schedule(
                new java.util.TimerTask() {
                    @Override
                    public void run() {
                        monster.setFill(img);
                        t.cancel();
                    }
                }, 50);
    }

    public void attackPlayer(Player player) {
        if (this.getBoundsInParent().intersects(player.getBoundsInParent())) {
            if (player.getIsAggressive()) {
                this.takeDamage(Player.getDamage());
            } else {
                player.takeDamage(this.damage);
            }
        }
    }

    public int getHealth() {
        return health;
    }

    public void setHealth(int health) {
        this.health = health;
    }

    public int getDamage() {
        return damage;
    }

    public void setDamage(int damage) {
        this.damage = damage;
    }

    public void setAlive(boolean alive) {
        this.alive = alive;
    }

    public boolean isAlive() {
        return this.alive;
    }

    public boolean isItemDropAvailable() {
        return itemDropAvailable;
    }


    public void setItemDropAvailable(boolean itemDropAvailable) {
        this.itemDropAvailable = itemDropAvailable;
    }
}
```
### Player.java
Player entity class with details like movement, weapons, health, etc. Player can move around, pick up weapons/potions, deal damage to monsters, take damage, and navigates rooms.
```
package dungeoncrawler.entity;

import dungeoncrawler.Controller;
import dungeoncrawler.entity.potion.Potion;
import dungeoncrawler.entity.potion.HealthPotion;
import dungeoncrawler.entity.potion.AttackPotion;
import dungeoncrawler.entity.potion.ZoomPotion;
import javafx.scene.image.Image;
import javafx.scene.paint.Color;
import javafx.scene.paint.ImagePattern;
import javafx.scene.shape.Rectangle;

import java.io.FileInputStream;
import java.io.FileNotFoundException;

public class Player extends Rectangle {
    public static final int ORIGINAL_HEALTH = 20;
    public static final int ORIGINAL_SPEED = 7;
    private static int MONSTERS_KILLED = 0;
    private static int POTIONS_DRANK = 0;
    private static int ITEMS_PURCHASED = 0;
    private static int health = ORIGINAL_HEALTH;
    private static int damageModifier = 0; //Tracks bonuses to player damage
    private static int damage;
    private static final Weapon[] WEAPON_INVENTORY = {new Weapon("Shortsword", 1),
                                                      new Weapon("Bludgeon", 2),
                                                      new Weapon("Greatsword", 3)};
    private static final Potion[] POTION_INVENTORY = {new HealthPotion(),
                                                      new AttackPotion(),
                                                      new ZoomPotion()};
    //alter this to change potion quantity
    private static final int[] INVENTORY_QUANTITY = {0, 0, 0, 1, 1, 1};
    private static Weapon currentWeapon;
    private Rectangle weapon;
    private boolean goNorth;
    private boolean goSouth;
    private boolean goEast;
    private boolean goWest;
    private static boolean alive = true;
    private boolean isAggressive;
    private static int speed = ORIGINAL_SPEED;

    public Player(double x, double y, int width, int height, Weapon startingWeapon) {
        super(x, y, width, height);
        this.setVisible(true);
        this.setFill(Color.RED);
        this.isAggressive = true;
        this.setId("player");
        if (currentWeapon == null) {
            currentWeapon = startingWeapon;
        }
        damage = damageModifier + currentWeapon.getDamage();
    }


    public Rectangle getWeaponSprite() {
        weapon = new Rectangle(this.getWidth(), this.getHeight());
        if (Player.getCurrentWeapon().getName().equals("Bludgeon")) {
            try {
                weapon.setFill(new ImagePattern(new Image(new FileInputStream(
                        System.getProperty("user.dir") + "\\res\\bludgeon.png"))));
            } catch (FileNotFoundException exception) {
                System.out.println("Bludgeon image not found " + exception);
            }
        } else if (Player.getCurrentWeapon().getName().equals("Greatsword")) {
            try {
                weapon.setFill(new ImagePattern(new Image(new FileInputStream(
                        System.getProperty("user.dir") + "\\res\\greatsword.png"))));
            } catch (FileNotFoundException exception) {
                System.out.println("Greatsword image not found " + exception);
            }
        } else {
            try {
                weapon.setFill(new ImagePattern(new Image(new FileInputStream(
                        System.getProperty("user.dir") + "\\res\\shortsword.png"))));
            } catch (FileNotFoundException exception) {
                System.out.println("Shortsword image not found " + exception);
            }
        }
        weapon.setX(this.getX() + 25);
        weapon.setY(this.getY());
        return (weapon);
    }

    public void move(int height, int width) {

        int dx = 0;
        int dy = 0;
        if (goNorth) {
            dy = -speed;
        }
        if (goWest) {
            dx = -speed;
        }
        if (goSouth) {
            dy = speed;
        }
        if (goEast) {
            dx = speed;
        }

        double newX = this.getX() + dx;
        double newY = this.getY() + dy;
        if (newY < 0) {
            newY = 0;
        }
        if (newX < 0) {
            newX = 0;
        }
        if (newY + this.getHeight() > height) {
            newY = height - this.getHeight();
        }
        if (newX + this.getWidth() > width) {
            newX = width - this.getWidth();
        }

        this.setX(newX);
        if (this.weapon != null) {
            weapon.setX(newX + 25);
            this.setY(newY);
            weapon.setY(newY);
        }
    }

    public void takeDamage(int damageCount) {
        health -= damageCount;
        if (health <= 0) {
            alive = false;
        }
    }

    public static void updateWeapon(Weapon newWeapon) {
        currentWeapon = newWeapon;
    }

    public static void resetStats() {
        Player.health = ORIGINAL_HEALTH;
        Player.speed = ORIGINAL_SPEED;
        Player.INVENTORY_QUANTITY[3] = 1;
        Player.INVENTORY_QUANTITY[4] = 1;
        Player.INVENTORY_QUANTITY[5] = 1;
        Player.INVENTORY_QUANTITY[0] = 0;
        Player.INVENTORY_QUANTITY[1] = 0;
        Player.INVENTORY_QUANTITY[2] = 0;
        Controller.setGold(100 - 25 * Controller.getDifficulty().ordinal());
    }
    public static Weapon[] getWeaponInventory() {
        return WEAPON_INVENTORY;
    }

    public static Potion[] getPotionInventory() {
        return POTION_INVENTORY;
    }

    public static int[] getInventoryQuantity() {
        return INVENTORY_QUANTITY;
    }

    public static Weapon getCurrentWeapon() {
        return currentWeapon;
    }

    public static boolean isAlive() {
        return alive;
    }

    public static void setHealth(int newHealth) {
        health = newHealth;
    }

    public static void setIsAlive(boolean isAlive) {
        alive = isAlive;
    }

    public void setGoNorth(boolean goNorth) {
        this.goNorth = goNorth;
    }

    public void setGoSouth(boolean goSouth) {
        this.goSouth = goSouth;
    }

    public void setGoEast(boolean goEast) {
        this.goEast = goEast;
    }

    public void setGoWest(boolean goWest) {
        this.goWest = goWest;
    }

    public static int getDamage() {
        return damage;
    }

    public static void setDamage(int newDamage) {
        damage = newDamage;
    }

    public void setIsAggressive(boolean isAggressive) {
        this.isAggressive = isAggressive;
    }

    public boolean getIsAggressive() {
        return this.isAggressive;
    }

    public static int getHealth() {
        return health;
    }

    public static int getSpeed() {
        return speed;
    }

    public static void setSpeed(int speed) {
        Player.speed = speed;
    }

    public static int getDamageModifier() {
        return damageModifier;
    }

    public static void setDamageModifier(int newDamageMod) {
        damageModifier = newDamageMod;
    }

    public static void killMonster() {
        MONSTERS_KILLED++;
    }

    public static int getMonstersKilled() {
        return MONSTERS_KILLED;
    }

    public static void drinkPotion() {
        POTIONS_DRANK++;
    }

    public static int getPotionsDrank() {
        return POTIONS_DRANK;
    }

    public static void purchaseItem() {
        ITEMS_PURCHASED++;
    }

    public static int getItemsPurchased() {
        return ITEMS_PURCHASED;
    }


}
```

## Spotifind
Developed an application called SpotiFind with a group to create a Spotify playlist of similar-sounding songs from a given song. Used Spotify's API to access each song's features, including tempo, key, danceability, etc. Utilized PCA to reduce dimensionality and find the 3 directions that most correlated with feature similarity between songs. To group the songs, implemented three different methods for comparison; DBScan, Naive Bayes, and K-nearest neighbors.
### spotify_data_retrieval.py
retrieving songs and their features using Spotify API
```
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials

import numpy as np
import pandas as pd

cid = ''
secret = ''

client_credentials_manager = SpotifyClientCredentials(client_id = cid, client_secret=secret)
sp = spotipy.Spotify(client_credentials_manager = client_credentials_manager)

# Spotify Song URL for comparison
search_artist = 'example artist'
search_track = 'example track'


# Arrays to titles and artists
song_titles = []
song_artists = []
artist_ids = []
song_years = []
song_ids = []


# Arrays of data features
song_danceability = []
song_energy = []
song_keys = []
song_loudness = []
song_modes = []
song_speechiness = []
song_acousticness = []
song_instrumentalness = []
song_liveness = []
song_valence = []
song_tempo = []
song_duration_ms = []
song_genre = []


nulls = []

# Parameters for api retrieval
# number of years back
year_range = 10
# number of songs
num_songs = 1000
query_limit = 50


def find_features(ids):
    song_features = sp.audio_features(ids)
    for j in range(len(song_features)):
        features = song_features[j]
        if (features == None):
            nulls.append((year * num_songs) + i + j - 1)
            song_danceability.append(None)
            song_energy.append(None)
            song_keys.append(None)
            song_loudness.append(None)
            song_modes.append(None)
            song_speechiness.append(None)
            song_acousticness.append(None)
            song_instrumentalness.append(None)
            song_liveness.append(None)
            song_valence.append(None)
            song_tempo.append(None)
            song_duration_ms.append(None)

        else:
            song_danceability.append(features['danceability'])
            song_energy.append(features['energy'])
            song_keys.append(features['key'])
            song_loudness.append(features['loudness'])
            song_modes.append(features['mode'])
            song_speechiness.append(features['speechiness'])
            song_acousticness.append(features['acousticness'])
            song_instrumentalness.append(features['instrumentalness'])
            song_liveness.append(features['liveness'])
            song_valence.append(features['valence'])
            song_tempo.append(features['tempo'])
            song_duration_ms.append(features['duration_ms'])

# Main loop
for year in range(year_range):
  search_year = 'year:20' + str(21 - year)
  print("Getting songs from", search_year)
  for i in range(0, num_songs, query_limit):
    track_results = sp.search(q=search_year, type='track', limit=query_limit, offset=i)
    tracks = track_results['tracks']['items']
    for j in range(len(tracks)):
      track = tracks[j]
      song_titles.append(track['name'])
      song_artists.append(track['artists'][0]['name'])
      artist_ids.append(track['artists'][0]['id'])
      song_years.append(track['album']['release_date'][0:4])
      song_ids.append(track['id'])


    find_features(song_ids[year*num_songs + i: year*num_songs + i +query_limit])


print("Getting Genres...")
for i in range(0, len(artist_ids), 20):
    curr_artists = sp.artists(artist_ids[i:i+20])['artists']
    for j in range(len(curr_artists)):
        if (curr_artists[j]['genres'] == []):
            song_genre.append("none")
        else:
            # print(curr_artists[j]['genres'][0])
            song_genre.append(curr_artists[j]['genres'][0])

# Adds requested song
# song_results = sp.search(q='artist:' + search_artist + ' track:' + search_track, type='track')
# search_track = song_results['tracks']['items'][0]
# song_titles.append(search_track['name'])
# song_artists.append(search_track['artists'][0]['name'])
# song_years.append(search_track['album']['release_date'][0:4])
# song_ids.append(search_track['id'])
# find_features([search_track['id']])
    

# data dictionary
data = {'titles': song_titles, 'artists': song_artists, 'artist id': artist_ids, 
        'year':song_years, 'ids': song_ids,

        'danceability': song_danceability, 'energy': song_energy,
        'key': song_keys, 'loudness': song_loudness, 'mode':song_modes,
        'speechiness': song_speechiness, 'acousticness': song_acousticness,
        'instrumentalness': song_instrumentalness, 'liveness': song_liveness,
        'valence': song_valence, 'tempo': song_tempo, 
        'duration_ms': song_duration_ms, 
        'genre': song_genre}

df = pd.DataFrame(data)
df = df.drop(df.index[nulls])
df = df.drop_duplicates(keep='first')
df = df.dropna(how='all')
df = df.drop_duplicates(subset=['titles', 'duration_ms'])

df.to_csv("spotify_api.csv")

print("Done!")

```
### PCA
reduce dimensionalities of features to 5 most important directions
```
# PCA
import numpy as np
import pandas as pd
from sklearn.decomposition import PCA

df = pd.read_csv('spotify_api (1).csv', engine='python')
features = list(df)[5:]
df_red = df.drop(['Unnamed: 0', 'titles', 'artists', 'year', 'ids'], axis=1)
array = df_red.to_numpy()
print(array)

pca = PCA(n_components=5)
reduced = pca.fit_transform(array)

#identify most impactful features
print(pca.components_)
print(pca.components_.shape)
var = np.square(pca.components_)
var = np.sum(var, axis=0)
idx = np.argsort(var)
features = [features[i] for i in idx]
print(np.flip(features)) # list of features from most to least important

df['dir 1'] = reduced[:, 0]
df['dir 2'] = reduced[:, 1]
df['dir 3'] = reduced[:, 2]
df['dir 4'] = reduced[:, 3]
df['dir 5'] = reduced[:, 4]
df.to_csv("spotify_api_pca5.csv")

print("PCA Done!")
```
### KNN
classify songs using 20 nearest neighbors
```
# K-nearest neighbors
from sklearn.neighbors import NearestNeighbors
from matplotlib import pyplot as plt
df = pd.read_csv('spotify_api_pca.csv', engine='python')
df_pca = df[['dir 1','dir 2','dir 3']]
print(df_pca)
dataset = df_pca.to_numpy()

neighbors = NearestNeighbors(n_neighbors=20)
neighbors_fit = neighbors.fit(dataset)
distances, indices = neighbors_fit.kneighbors(dataset)

distances = np.sort(distances, axis=0)
distances = distances[4500:,1]
plt.plot(distances)
```

## Dialogflow Solution Webhook (zavvie)
Took in user input of (County, State) from Dialogflow, extracted parameters, passed them into zavvie API, reformatted output, and returned solutions in user’s requested location.
### Cloud Function
Use user input of location to call API and return solutions in location
```
const functions = require('@google-cloud/functions-framework');
const unirest = require('unirest');

// Here's where you set the entry point, it gets the request from Dialogflow CX 
// and it holds the response which will be sent back to Dialogflow 
functions.http('solutionhook', (req, res) => {
    var output = "";
    var userInput = req.body.sessionInfo.parameters.location.original;
    console.log(userInput);
    var county = userInput.substring(0, userInput.length - 4);
    var state = userInput.substring(userInput.length - 2, userInput.length);
    var apiResponse = unirest.post("https://api.zavvie.com/v2/api/eligible-programs");
        apiResponse.headers({
            'zavvie-api-key': 'CHANGE ME',
            'content-type' : 'application/json'
        });
        apiResponse.send({
            'client-type': 'seller',
            'locations': [{'state': state,'county': county}]
        });
        apiResponse.end(function(apiRes) {
            if(apiRes.error) {
                console.log("OOPS! error");
                console.log(JSON.stringify(apiRes));
            } else {
                output += "Solutions available for " + userInput + ":\n";
                for (var i in apiRes.body.data) {
                    if (apiRes.body.data[i] != null) {
                        output += "Solution: " + JSON.stringify(apiRes.body.data[i]["display_name"]) + "\n";
                        for (var j in apiRes.body.data[i]["programs"]) {
                            output += "- " + "Program provider: " + JSON.stringify(apiRes.body.data[i]["programs"][j]["program_name"]) + "\n";
                        }
                    }
                }
                console.log("SUCCESS! response: ");
                console.log(output);
                console.log("full data:");
                console.log(JSON.stringify(apiRes.body.data));
            }
            var response = {
                fulfillmentResponse: {
                    messages: [
                        {
                            text: {
                                text: [
                                    output // this is the message for the agent to return
                                ],
                            },
                        },
                    ],
                },
                sessionInfo: {
                    parameters: 
                    {
                        "success": true,
                    }
                }
            };

            res.status(200).json(response);
        });
});
```

## Dialogflow Text Embedding Webhook (zavvie)
Generated text embeddings for FAQs, then set up Matching Engine’s Approximate Nearest Neighbors with Vertex AI to use cosine distance for vector comparison. Passed in user’s FAQ input from Dialogflow, and used cloud function to translate the input to text embedding vector, call Matching Engine endpoint with vector as parameter, and select the closest vector returned from the endpoint. Checked to ensure that the distance is close enough to suggest similar enough meaning, selected the answer associated with the closest stored FAQ, passed the answer through Generative AI to add variation to the wording, and returned the altered answer.
### Cloud Function
Map user input to closest stored FAQ through text embedding and return answer with salt from Generative AI
```
import functions_framework

#dict mapping detected intents to answers
answers = {}

@functions_framework.http
def call_matching_engine(request):
    """HTTP Cloud Function.
    Args:
        request (flask.Request): The request object.
        <https://flask.palletsprojects.com/en/1.1.x/api/#incoming-request-data>
    Returns:
        The response text, or any set of values that can be turned into a
        Response object using `make_response`
        <https://flask.palletsprojects.com/en/1.1.x/api/#flask.make_response>.
    """

    import google.cloud.logging 
    client = google.cloud.logging.Client()
    client.setup_logging()
    import logging
    import json

    import google.cloud.aiplatform
    from google.cloud.aiplatform.matching_engine.matching_engine_index_endpoint import Namespace
    from vertexai.preview.language_models import TextEmbeddingModel
    import vertexai
    from vertexai.language_models import TextGenerationModel

    request_json = request.get_json(silent=True)
    request_args = request.args
    logging.info("mic check")
    userInput = request_json['text']
    logging.info(userInput)

    model = TextEmbeddingModel.from_pretrained("textembedding-gecko@001")
    embeddings = model.get_embeddings([userInput])[0].values
    logging.info(embeddings)

    similarity_engine = google.cloud.aiplatform.MatchingEngineIndexEndpoint(
        index_endpoint_name = "7444793231671296000",
        project = "intern-378317",
        location = "us-central1"
    )

    neighbors = similarity_engine.match(
        deployed_index_id="similarity_engine",
        queries=[embeddings],
        num_neighbors=4
    )

    closest = [{"id": neighbor.id, "distance": neighbor.distance} for neighbor in neighbors[0]]
    logging.info(closest)
    closestIntent = closest[0]["id"]
    closestIntent = closestIntent[0:closestIntent.rindex('-')]
    closestDistance = closest[0]["distance"]
    logging.info(closestIntent)
    logging.info("distance: " + str(closestDistance))
    if closestDistance < 0.2:
        output = answers[closestIntent]
        logging.info(output)

        vertexai.init(project="intern-378317", location="us-central1")
        parameters = {
            "temperature": 0.5,
            "max_output_tokens": 256,
            "top_p": 0.8,
            "top_k": 40
        }
        model = TextGenerationModel.from_pretrained("text-bison@001")
        modelResponse = model.predict(
            """Reword the following sentence(s) in a slightly different way:\n""" + output,
            **parameters
        )

        response = {"fulfillment_response": {"messages": [{"text": {"text": [modelResponse.text]}}]}}
    else:
        response = {"fulfillment_response": {"messages": [{"text": {"text": ["Sorry, I'm not sure if I know the answer to that."]}}]}}
    return (response)
```
### Embeddings pipeline
Generate embeddings for stored intents
```
import kfp
import kfp.dsl as dsl

from kfp import compiler
from kfp.dsl import Dataset, Input, Output

from typing import List

@dsl.component(
    base_image="python:3.11",
    packages_to_install=["google-cloud-aiplatform", "appengine-python-standard"],
)
def generate_embeddings(markdown_gcs_dir: str):
    import json
    import os
    from vertexai.preview.language_models import TextEmbeddingModel

    model = TextEmbeddingModel.from_pretrained("textembedding-gecko@001")

    local_dir = markdown_gcs_dir.replace("gs://", "/gcs/") + "embed-prompts/intents/"
    # Loop over files in the GCS directory
    data = []
    for filename in os.listdir(local_dir):
        print(filename)
        temp = os.path.join(local_dir, filename)  
        if os.path.isdir(temp):
            print(temp + " is dir!")  
            trainFolder = os.path.join(temp, 'trainingPhrases')
            jsonFile = os.path.join(trainFolder, 'en.json')
            with open(jsonFile) as f:
                fullJson = json.load(f)
                for i in range(len(fullJson["trainingPhrases"])):
                    text = ""
                    for j in fullJson["trainingPhrases"][i]["parts"]:
                        text += j["text"]           
                    embeddings = model.get_embeddings([text])
                    for embedding in embeddings:
                        data.append(
                            json.dumps(
                                {
                                    "id": filename + "-" + str(i),
                                    "embedding": embedding.values
                                }
                            )
                        )

    with open(markdown_gcs_dir.replace('gs://', '/gcs/') + "engine/data.json", 'w') as f:
        f.write("\n".join(data))


@dsl.pipeline(name="similarity-engine-posts")
def transcript_extraction():
    generate_embeddings(markdown_gcs_dir="gs://[BUCKET NAME HERE]/posts/")

compiler.Compiler().compile(transcript_extraction, "pipeline.yaml")
```
### Answer pipeline
Create json with answers mapped to intents
```
import kfp
import kfp.dsl as dsl

from kfp import compiler
from kfp.dsl import Dataset, Input, Output

from typing import List

@dsl.component(
    base_image="python:3.11",
    packages_to_install=["google-cloud-aiplatform", "appengine-python-standard"],
)
def generate_answers(markdown_gcs_dir: str):
    import json
    import os

    local_dir = markdown_gcs_dir.replace("gs://", "/gcs/") + "embed-prompts/flows/"
    # Loop over files in the GCS directory
    data = dict()
    for filename in os.listdir(local_dir):
        print(filename)
        temp = os.path.join(local_dir, filename)  
        if os.path.isdir(temp):
            print(temp + " is dir!")  
            jsonFile = os.path.join(temp, filename + '.json')
            with open(jsonFile) as f:
                fullJson = json.load(f)
                for i in range(len(fullJson["transitionRoutes"])):
                    intent = fullJson["transitionRoutes"][i]["intent"]
                    answer = ""
                    if "messages" in fullJson["transitionRoutes"][i]["triggerFulfillment"] and len(fullJson["transitionRoutes"][i]["triggerFulfillment"]["messages"]) > 0:
                        for j in fullJson["transitionRoutes"][i]["triggerFulfillment"]["messages"][0]["payload"]["richContent"][0]:
                            if "subtitle" in j:
                                answer += j["subtitle"] + "\n"
                            data[intent] = answer  
    with open(markdown_gcs_dir.replace('gs://', '/gcs/') + "engine/answers.json", 'w') as f:
        json.dump(data, f)


@dsl.pipeline(name="similarity-engine-posts")
def transcript_extraction():
    generate_answers(markdown_gcs_dir="gs://[BUCKET NAME HERE]/posts/")

compiler.Compiler().compile(transcript_extraction, "answers.yaml")
```
