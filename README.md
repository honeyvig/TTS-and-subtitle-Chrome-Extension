# TTS-and-subtitle-Chrome-Extension
create a chrom extension that demonstrates the core functionality of a text-to-speech (TTS) and auto-translated subtitle system for web pages, specifically focusing on Spanish text being translated into English subtitles. This might be growing into a larger project later on, but the initial goal is to focus on a smooth and functional core experience, with a local application. The focus of this phase will be on the core functions of processing text from a Spanish web page, converting the text to speech, and displaying English subtitles in real-time. The resulting application does not need to be a full-fledged Chrome extension initially, but should lay the foundation for it.
Key Responsibilities:
●
Develop a chrome extension that can be installed and run on my computer.
●
Implement functionality to extract text from a given URL of a web page written in Spanish.
●
Integrate Text-to-Speech (TTS) API to read the extracted Spanish text aloud.
●
Implement translated English subtitles based on the Spanish text.
●
Display the English subtitles clearly on screen, synchronized with the audio.
●
Ensure the application works smoothly and efficiently, with an emphasis on a clear and intuitive user experience.
●
Focus on functionality and performance rather than advanced UI features for this initial stage.
●
Provide the source code, and relevant comments for clarity

Important note! TTS needs to be in the original text language (Spanish), Subtitles need to be in English translated from the original Spanish text
-----------
To create a Chrome extension that implements a basic Text-to-Speech (TTS) and real-time translation of subtitles for Spanish web pages into English, we will need to:

    Extract text content from a webpage.
    Use a TTS engine to read the text in Spanish.
    Use a translation API (e.g., Google Translate) to translate the extracted Spanish text into English and display subtitles in sync with the TTS.

Here’s a breakdown of how you can implement this Chrome extension:
Step 1: Setup the Manifest File (manifest.json)

This will define the necessary permissions and content scripts.

{
  "manifest_version": 3,
  "name": "TTS & Translation for Spanish Text",
  "description": "Translate and read aloud Spanish text on web pages with subtitles in English.",
  "version": "1.0",
  "permissions": ["activeTab", "storage"],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html"
  },
  "host_permissions": [
    "https://translate.googleapis.com/*"
  ]
}

Step 2: HTML Popup (popup.html)

Create a simple UI for starting and stopping the TTS and subtitle system.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Spanish TTS & Translation</title>
  <style>
    body { font-family: Arial, sans-serif; padding: 10px; }
    button { padding: 10px; }
  </style>
</head>
<body>
  <h3>Text-to-Speech and Translation</h3>
  <button id="startButton">Start TTS & Translation</button>
  <button id="stopButton">Stop</button>

  <script src="popup.js"></script>
</body>
</html>

Step 3: Popup JS (popup.js)

Control the start and stop behavior of the TTS and translation system.

document.getElementById('startButton').addEventListener('click', () => {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    chrome.scripting.executeScript({
      target: { tabId: tabs[0].id },
      function: startTTSAndTranslation,
    });
  });
});

document.getElementById('stopButton').addEventListener('click', () => {
  chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
    chrome.scripting.executeScript({
      target: { tabId: tabs[0].id },
      function: stopTTSAndTranslation,
    });
  });
});

function startTTSAndTranslation() {
  chrome.runtime.sendMessage({ action: "start" });
}

function stopTTSAndTranslation() {
  chrome.runtime.sendMessage({ action: "stop" });
}

Step 4: Background JS (background.js)

Manage the starting and stopping of the TTS system and translation.

let isRunning = false;
let interval;

chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === "start") {
    if (!isRunning) {
      isRunning = true;
      startProcessing();
    }
  } else if (request.action === "stop") {
    if (isRunning) {
      isRunning = false;
      clearInterval(interval);
    }
  }
});

function startProcessing() {
  interval = setInterval(() => {
    chrome.tabs.query({ active: true, currentWindow: true }, (tabs) => {
      chrome.scripting.executeScript({
        target: { tabId: tabs[0].id },
        function: processTextAndSpeech
      });
    });
  }, 1000); // Adjust interval to read at a suitable pace
}

function processTextAndSpeech() {
  const pageText = extractText();
  if (pageText) {
    const translatedText = translateText(pageText);
    speakText(pageText, translatedText);
  }
}

function extractText() {
  // Extract all text from the body of the page
  const text = document.body.innerText;
  return text;
}

async function translateText(text) {
  // Use Google Translate API to translate text to English
  const response = await fetch(`https://translate.googleapis.com/translate_a/single?client=gtx&sl=es&tl=en&dt=t&q=${encodeURIComponent(text)}`);
  const data = await response.json();
  return data[0][0][0]; // Return translated text
}

function speakText(originalText, translatedText) {
  // Use the SpeechSynthesis API to speak in Spanish
  const utterance = new SpeechSynthesisUtterance(originalText);
  utterance.lang = "es-ES"; // Spanish
  speechSynthesis.speak(utterance);

  // Display English subtitles
  displaySubtitles(translatedText);
}

function displaySubtitles(translatedText) {
  const subtitleDiv = document.getElementById('subtitleDiv');
  if (!subtitleDiv) {
    const newSubtitleDiv = document.createElement('div');
    newSubtitleDiv.id = 'subtitleDiv';
    newSubtitleDiv.style.position = 'fixed';
    newSubtitleDiv.style.bottom = '10px';
    newSubtitleDiv.style.left = '50%';
    newSubtitleDiv.style.transform = 'translateX(-50%)';
    newSubtitleDiv.style.backgroundColor = 'rgba(0, 0, 0, 0.7)';
    newSubtitleDiv.style.color = 'white';
    newSubtitleDiv.style.padding = '10px';
    newSubtitleDiv.style.fontSize = '20px';
    document.body.appendChild(newSubtitleDiv);
    subtitleDiv = newSubtitleDiv;
  }

  subtitleDiv.innerText = translatedText;
}

Step 5: Content Script (content.js)

Extract text content from the webpage.

function extractTextFromPage() {
  // You can extract specific content based on the structure of the webpage.
  return document.body.innerText;
}

Step 6: Testing and Debugging

    Load the Extension:
        Go to chrome://extensions/.
        Enable "Developer mode."
        Click "Load unpacked" and select the directory of your extension.
    Testing the TTS & Translation:
        Open any Spanish web page.
        Click the extension icon and press "Start TTS & Translation."
        The page's text will be read out in Spanish, and subtitles will appear in English.

Further Considerations

    Language Detection: Ensure that the page is in Spanish, as this extension assumes the text is in Spanish.
    Performance: For larger pages, optimize the translation and TTS processing to prevent high CPU usage.
    Real-Time Synchronization: To sync subtitles with the speech more effectively, you might need to break the text into smaller chunks and display each chunk as it is being spoken.
