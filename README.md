# Word Guessing Game 
## Getting introduced with SpeechRecognition Library
- SpeechRecognition library is a wrapper of Google web Speech API and some other speech API. And the good news is one can use it without signing up for any of these services.
- SpeechRecognition is compatible with python 2.6, 2.7 and 3.3+. But it's better to use python 3.3+ as no additional installation steps required for python 3.3+.

### Installing SpeechRecognition 
```bash 
$ pip install SpeechRecognition
```
Varify if installed:
``` python
import speech_recognition as sr
sr.__version__
```
It should output the version of the library if perfectly Installed.

Now we will be needing 2 additional python packages.
- random
- time
## Game Description:
- A list of 6 Fruit's name will be displayed.
- A random fruit will be selected from the list.
- So, to win The gamer will have to guess that selected fruit name in 3 guesses.  
## Building The Word Gussing Game

- open jupyter notebook and import required Libraries
```Python
import random
import time
import speech_recognition as sr
```
### Writing speech recognizer function:
this fuction will transcribe speech from microphone.

- Add following condition in the function to check thar the recognizer argument is appropriate type.

```python
if not isinstance(recognizer, sr.Recognizer):
        raise TypeError("`recognizer` must be `Recognizer` instance")
```
- Add following condition in the function to check thar the microphone argument is appropriate type.

```Python
if not isinstance(microphone, sr.Microphone):
        raise TypeError("`microphone` must be `Microphone` instance")
```
- In real world we can not expect audio with no noice. So, to adjust the recognizer sensitivity to ambient noise and record audio :
```Python
 with microphone as source:
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)
```
- It's not expected that the recognizer will return the expected result always. The API can be unreachable or it can fail to recognize any speech so to identify this errors we need a response object.
```Python
 response = {
        "success": True,
        "error": None,
        "transcription": None
    }
```
- Now, for identifying the success and errors and saving the response:
```Python
try:
        response["transcription"] = recognizer.recognize_google(audio)
    except sr.RequestError:
        # API was unreachable or unresponsive
        response["success"] = False
        response["error"] = "API unavailable"
    except sr.UnknownValueError:
        # speech was unintelligible
        response["error"] = "Unable to recognize speech"
```
- And the response object will be returned as the function result.
- The whole function:
```Python
def recognize_speech_from_mic(recognizer, microphone):

    if not isinstance(recognizer, sr.Recognizer):
        raise TypeError("`recognizer` must be `Recognizer` instance")

    if not isinstance(microphone, sr.Microphone):
        raise TypeError("`microphone` must be `Microphone` instance")

    with microphone as source:
        recognizer.adjust_for_ambient_noise(source)
        audio = recognizer.listen(source)

    
    response = {
        "success": True,
        "error": None,
        "transcription": None
    }

  
    try:
        response["transcription"] = recognizer.recognize_google(audio)
    except sr.RequestError:
        response["success"] = False
        response["error"] = "API unavailable"
    except sr.UnknownValueError:
        
        response["error"] = "Unable to recognize speech"

    return response
```

### Writing the main function:
As it's a testing Project we are not using any dataset just using a array of fruit's name.

- Initialize recognizer and microphone instances
```Pyhton
recognizer = sr.Recognizer()
    microphone = sr.Microphone()
```
- pick a random word from dataset. This is where we need the Random package we have already imported.
```
word = random.choice(WORDS)
```
- We need a sleep function for 3 seconds so that microphone can listen.
```
time.sleep(3)
```
- Now we will iteret for guesses and
  for each time microphone will listen a word and give the verdits.
  there will be three types of verdict:
  - if success status is true and the word is same as the guessed word then verdict : Correct! You win!
  - if success status is false then verdict:  'I didn't catch that. What did you say?' and in this case the function will listen again and a extra chance will be given
  - if if success status is true and the word is not the guessed word and guess left then verdict : Incorrect. Try again.
  - if if success status is true and the word is not the guessed word and no guess left then verdict : Sorry, you lose!\nI was thinking of

So the main function will be:

```Python
if __name__ == "__main__":
   
    WORDS = ["apple", "banana", "grape", "orange", "mango", "lemon"]
    NUM_GUESSES = 3
    PROMPT_LIMIT = 5

  
    recognizer = sr.Recognizer()
    microphone = sr.Microphone()


    word = random.choice(WORDS)

  
    instructions = (
        "I'm thinking of one of these words:\n"
        "{words}\n"
        "You have {n} tries to guess which one.\n"
    ).format(words=', '.join(WORDS), n=NUM_GUESSES)

   
    print(instructions)
    time.sleep(3)

    for i in range(NUM_GUESSES):
     
        for j in range(PROMPT_LIMIT):
            print('Guess {}. Speak!'.format(i+1))
            guess = recognize_speech_from_mic(recognizer, microphone)
            if guess["transcription"]:
                break
            if not guess["success"]:
                break
            print("I didn't catch that. What did you say?\n")

        # if there was an error, stop the game
        if guess["error"]:
            print("ERROR: {}".format(guess["error"]))
            break

        # show the user the transcription
        print("You said: {}".format(guess["transcription"]))

        # determine if guess is correct and if any attempts remain
        guess_is_correct = guess["transcription"].lower() == word.lower()
        user_has_more_attempts = i < NUM_GUESSES - 1

        # determine if the user has won the game
        # if not, repeat the loop if user has more attempts
        # if no attempts left, the user loses the game
        if guess_is_correct:
            print("Correct! You win!".format(word))
            break
        elif user_has_more_attempts:
            print("Incorrect. Try again.\n")
        else:
            print("Sorry, you lose!\nI was thinking of '{}'.".format(word))
            break
```

I've done this by following [this Link](https://realpython.com/python-speech-recognition/)