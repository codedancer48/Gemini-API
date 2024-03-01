<p align="center">
    <img src="https://www.gstatic.com/lamda/images/gemini_wordmark_landing_page_238102af073d0ae2763aa5.svg" alt="Gemini Workmark" align="center">
</p>

# <img src="https://www.gstatic.com/lamda/images/favicon_v1_150160cddff7f294ce30.svg" width="35px" alt="Gemini Icon" /> Gemini-API

A reverse-engineered asynchronous python wrapper for [Google Gemini](https://gemini.google.com) (formerly Bard).

## Features

- **ImageFx Support** - Supports retrieving images generated by ImageFx, Google's latest AI image generator.
- **Extension Support** - Supports generating contents with [Gemini extensions](https://gemini.google.com/extensions) on, like YouTube and Gmail.
- **Classified Outputs** - Auto categorizes texts, web images and AI generated images from the response.
- **Official Flavor** - Provides a simple and elegant interface inspired by [Google Generative AI](https://ai.google.dev/tutorials/python_quickstart)'s official API.
- **Asynchronous** - Utilizes `asyncio` to run generating tasks and return outputs efficiently.

## Installation

```bash
pip install gemini_webapi
```

## Authentication

- Go to <https://gemini.google.com> and login with your Google account
- Press F12 for web inspector, go to `Network` tab and refresh the page
- Click any request and copy cookie values of `__Secure-1PSID` and `__Secure-1PSIDTS`

Note: `__Secure-1PSIDTS` could get expired frequently if the Google account is actively used elsewhere, especially when visiting <https://gemini.google.com> directly. It's recommended to use a separate Google account if you are builing a keep-alive service with this package.

## Usage

### Initialization

Import required packages and initialize a client with your cookies obtained from the previous step.

```python
import asyncio
from gemini_webapi import GeminiClient

# Replace "COOKIE VALUE HERE" with your actual cookie values
Secure_1PSID = "COOKIE VALUE HERE"
Secure_1PSIDTS = "COOKIE VALUE HERE"

async def main():
    client = GeminiClient(Secure_1PSID, Secure_1PSIDTS, proxy=None)
    await client.init(timeout=30, auto_close=False, close_delay=300)

asyncio.run(main())
```

Note: `auto_close` and `close_delay` are optional arguments for automatically closing the client after a certain period of inactivity. This feature is disabled by default. In a keep-alive service like chatbot, it's recommended to set `auto_close` to `True` combined with reasonable seconds of `close_delay` for better resource management.

### Generate contents from text inputs

Ask a one-turn quick question by calling `GeminiClient.generate_content`.

```python
async def main():
    response = await client.generate_content("Hello World!")
    print(response.text)

asyncio.run(main())
```

Note: simply use `print(response)` to get the same output if you just want to see the response text

### Conversations across multiple turns

If you want to keep conversation continuous, please use `GeminiClient.start_chat` to create a `ChatSession` object and send messages through it. The conversation history will be automatically handled and get updated after each turn.

```python
async def main():
    chat = client.start_chat()
    response1 = await chat.send_message("Briefly introduce Europe")
    response2 = await chat.send_message("What's the population there?")
    print(response1.text, response2.text, sep="\n\n----------------------------------\n\n")

asyncio.run(main())
```

### Retrieve images in response

Images in the API's output are stored as a list of `Image` objects. You can access the image title, URL, and description by calling `image.title`, `image.url` and `image.alt` respectively.

```python
async def main():
    response = await client.generate_content("Send me some pictures of cats")
    for image in response.images:
        print(image, "\n\n----------------------------------\n")

asyncio.run(main())
```

### Generate image with ImageFx

In February 2022, Google introduced a new AI image generator called ImageFx and integrated it into Gemini. You can ask Gemini to generate images with ImageFx simply by natural language.

**Important**: Google has some limitations on the image generation feature in Gemini, so its availability could be different per region/account. Here's a summary copied from [official documentation](https://support.google.com/gemini/answer/14286560) (as of February 15th, 2024):

>Image generation in Gemini Apps is available in most countries, except in the European Economic Area (EEA), Switzerland, and the UK. It’s only available for **English prompts**.
>
>This feature’s availability in any specific Gemini app is also limited to the supported languages and countries of that app.
>
>For now, this feature isn’t available to users under 18.

```python
async def main():
    response = await client.generate_content("Generate some pictures of cats")
    for image in response.images:
        print(image, "\n\n----------------------------------\n")

asyncio.run(main())
```

Note: by default, when asked to send images (like the previous example), Gemini will send images fetched from web instead of generating images with AI model, unless you specifically require to "generate" images in your prompt. In this package, web images and generated images are treated differently as `WebImage` and `GeneratedImage`, and will be automatically categorized in the output.

### Save images to local files

You can save images returned from Gemini to local files under `/temp` by calling `Image.save()`. Optionally, you can specify the file path and file name by passing `path` and `filename` arguments to the function and skip images with invalid file names by passing `skip_invalid_filename=True`. Works for both `WebImage` and `GeneratedImage`.

```python
async def main():
    response = await client.generate_content("Generate some pictures of cats")
    for i, image in enumerate(response.images):
        await image.save(path="temp/", filename=f"cat_{i}.png", verbose=True)

asyncio.run(main())
```

### Generate contents with Gemini extensions

**Important**: To access Gemini extensions in API, you must activate them on the [Gemini website](https://gemini.google.com/extensions) first. Same as image generation, Google also has limitations on the availability of Gemini extensions. Here's a summary copied from [official documentation](https://support.google.com/gemini/answer/13695044) (as of February 18th, 2024):

>To use extensions in Gemini Apps:
>
>Sign in with your personal Google Account that you manage on your own. Extensions, including the Google Workspace extension, are currently not available to Google Workspace accounts for school, business, or other organizations.
>
>Have Gemini Apps Activity on. Extensions are only available when Gemini Apps Activity is turned on.
>
>Important: For now, extensions are available in **English, Japanese, and Korean** only.

Note: for the last item, instead of region requirement, it actually only requires your Google account's **preferred language** to be set to one of the supported languages. You can change your language settings [here](https://myaccount.google.com/language).

After activating extensions for your account, you can access them in your prompts either by natural language or by starting your prompt with "@" followed by the extension keyword.

```python
async def main():
    response1 = await client.generate_content("@Gmail What's the latest message in my mailbox?")
    print(response1, "\n\n----------------------------------\n")

    response2 = await client.generate_content("@Youtube What's the lastest activity of Taylor Swift?")
    print(response2, "\n\n----------------------------------\n")

asyncio.run(main())
```

### Check and switch to other answer candidates

A response from Gemini usually contains multiple reply candidates with different generated contents. You can check all candidates and choose one to continue the conversation. By default, the first candidate will be chosen automatically.

```python
async def main():
    # Start a conversation and list all reply candidates
    chat = client.start_chat()
    response = await chat.send_message("Recommend a science fiction book for me.")
    for candidate in response.candidates:
        print(candidate, "\n\n----------------------------------\n")

    if len(response.candidates) > 1:
        # Control the ongoing conversation flow by choosing candidate manually
        new_candidate = chat.choose_candidate(index=1)  # Choose the second candidate here
        followup_response = await chat.send_message("Tell me more about it.")  # Will generate contents based on the chosen candidate
        print(new_candidate, followup_response, sep="\n\n----------------------------------\n\n")
    else:
        print("Only one candidate available.")

asyncio.run(main())
```

## References

[Google AI Studio](https://ai.google.dev/tutorials/ai-studio_quickstart)

[acheong08/Bard](https://github.com/acheong08/Bard)
