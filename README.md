مِيشِيلْ لُوْتِيْتُو ، اَلرَّجُلُ اَلَّذِي لَمْ يَكْتَفِ بِأَكْلِ اَلزُّجَاجِ وَالْمَعْدِنِ وَالْمَطَّاطِ ، بَلْ ذَهَبَ إِلَى أَبْعَدَ مِنْ ذَلِكَ بِكَثِيرٍ . هَلْ تَعْلَمُونَ أَنَّهُ أَكْلُ طَائِرَةٍ بِأَكْمَلِهَا ؟ نَعِمَ ، طَائِرَةٌ ! وَلَمْ يَتَوَقَّفْ عِنْدَ هَذَا اَلْحَدِّ ، فَقَدْ كَانَتْ اَلدَّرَّاجَاتُ وَشَفَرَاتُ اَلْحِلَاقَةِ جُزْءًا مِنْ قَائِمَةِ طَعَامِهِ أَيْضًا .# 11Labs Unleashed

Provides unlimited ElevenLabs API calls.


## Disclaimer!
This project is getting a bit too popular, be reasonable when creating fake accounts, if too many fake accounts are created, the 11Labs team will start investigating these fake accounts, and patch automated account creation with captcha (they did it but [#6](https://github.com/GaspardCulis/elevenlabs-unleashed/issues/6) showed me how to bypass it) or even remove free access to its services.


## Installation

### Dependencies

You need the [chromedriver](https://chromedriver.chromium.org/downloads) in your PATH.

### Pip installation

```bash
pip install git+https://github.com/GaspardCulis/elevenlabs-unleashed.git
```

## Usage

Create an account

```py
from elevenlabs_unleashed.account import create_account

username, password, api_key = create_account()
```

Automatic API key renewal

```py
from elevenlabs_unleashed.manager import ELUAccountManager
from elevenlabs import generate, set_api_key, play, api

eluac = ELUAccountManager(set_api_key, nb_accounts= 2) # Creates a queue of API keys
eluac.next() # First call will block the thread until keys are generated, and call set_api_key

def speak(self, message: str):
    try:
        audio = generate(
            text=message,
            voice="Josh", # I like this one
            model="eleven_multilingual_v1"
        )
    except elevenlabs.api.error.RateLimitError as e:
        print("[ElevenLabs] Maximum number of requests reached. Getting a new API key...")
        eluac.next() # Uses next API key in queue, should be instant as nb_accounts > 1, and will generate a new key in a background thread.
        speak(message)
        return

    print("[ElevenLabs] Starting the stream...")
    play(audio)
```

Full-on unlimited 11Labs API wrapper
    
```py
from elevenlabs_unleashed.tts import UnleashedTTS

tts = UnleashedTTS(nb_accounts=2, create_accounts_threads=2)
"""
Will automatically generate 2 accounts in 2 threads. Takes a few seconds.
"""

tts.speak("Hello world!", voice="Josh", model="eleven_multilingual_v1")
```

## How it works

11Labs Unleashed is basically just a web scraper (selenium) that creates unlimited 11Labs accounts programatically.

The `ELUAccountManager` stores an array of API keys populated in a FIFO queue manner. When calling *next()*, it returns the last API key in the queue (making sure it is not empty), and refills the queue, making the API key renewal instant after the first *next()* call as long as nb_accounts is greater than 1 (defaults to 2, more would be overkill).

The `UnleashedTTS` class is a wrapper around the ElevenLabs API, it automatically creates a given amount of 11Labs accounts and saves them in a userdata json file at initialisation. When calling *speak()* it will take the account with the higher API usage while still having enough characters left (11Labs bans your IP temporarly if you use too many accounts in a short period of time). At initialisation and after each *speak()* call, it will update each account's API usage (not saving it to the userdata json file).

You can run the account creation procedure with the browser visible by executing the python process with `DEBUG=1` environment variable.

## TODO

- Automatic account deletion when max API usage reached
- Less crappy Python code and better API
- Try not to get sued by 11Labs

## Notes

This library is very unstable and I guess won't work for long. It only relies on the fact that 11Labs account creation is easily bot-able. Also some minor [11Labs website](https://beta.elevenlabs.io/) changes might break my crappy web scraping.

If you find issues don't hesitate to submit a PR if you find a fix.

Using this code might temporarly ban your IP from using 11Labs API, refer to [this](https://help.elevenlabs.io/hc/en-us/articles/14129701265681-Why-am-I-receiving-information-about-unusual-activity-)

## Credits

Thanks to the ElevenLabs team for making the best multi-lingual TTS models in the world. But because the API pricing is such expensive, this library had to be done.

And so thanks to [Wikidepia](https://github.com/Wikidepia) for creating the [hektCaptcha-extension](https://github.com/Wikidepia/hektCaptcha-extension) allowing me to bypass the 11Labs captcha lol.
