- 👋 Hi, I’m @temerlan123
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...

def record_and_recognize_audio(*args: tuple):
    """
    Запись и распознавание аудио
    """
    with microphone:
        recognized_data = ""

        # регулирование уровня окружающего шума
        recognizer.adjust_for_ambient_noise(microphone, duration=2)

        try:
            print("Listening...")
            audio = recognizer.listen(microphone, 5, 5)

        except speech_recognition.WaitTimeoutError:
            print("Can you check if your microphone is on, please?")
            return

        # использование online-распознавания через Google 
        try:
            print("Started recognition...")
            recognized_data = recognizer.recognize_google(audio, language="ru").lower()

        except speech_recognition.UnknownValueError:
            pass

        # в случае проблем с доступом в Интернет происходит выброс ошибки
        except speech_recognition.RequestError:
            print("Check your Internet Connection, please")

        return recognized_dataconfig = {
    "intents": {
        "greeting": {
            "examples": ["привет", "здравствуй", "добрый день",
                         "hello", "good morning"],
            "responses": play_greetings
        },
        "farewell": {
            "examples": ["пока", "до свидания", "увидимся", "до встречи",
                         "goodbye", "bye", "see you soon"],
            "responses": play_farewell_and_quit
        },
        "google_search": {
            "examples": ["найди в гугл",
                         "search on google", "google", "find on google"],
            "responses": search_for_term_on_google
        },
    },
    "failure_phrases": play_failure_phrase
}

Такой вариант подойдёт тем, кто хочет натренировать ассистента на то, чтобы он отвечал на сложные фразы. Более того, здесь можно применить NLU-подход и создать возможность предугадывать намерение пользователя, сверяя их с теми, что уже есть в конфигурации.

Подробно этот способ мы его рассмотрим на 5 шаге данной статьи. А пока обращу ваше внимание на более простой вариант

2 способ

Можно взять упрощенный словарь, у которого в качестве ключей будет hashable-тип tuple (поскольку словари используют хэши для быстрого хранения и извлечения элементов), а в виде значений будут названия функций, которые будут выполняться. Для коротких команд подойдёт вот такой вариант:

commands = {
    ("hello", "hi", "morning", "привет"): play_greetings,
    ("bye", "goodbye", "quit", "exit", "stop", "пока"): play_farewell_and_quit,
    ("search", "google", "find", "найди"): search_for_term_on_google,
    ("video", "youtube", "watch", "видео"): search_for_video_on_youtube,
    ("wikipedia", "definition", "about", "определение", "википедия"): search_for_definition_on_wikipedia,
    ("translate", "interpretation", "translation", "перевод", "перевести", "переведи"): get_translation,
    ("language", "язык"): change_language,
    ("weather", "forecast", "погода", "прогноз"): get_weather_forecast,
}

Для его обработки нам потребуется дополнить код следующим образом:

def execute_command_with_name(command_name: str, *args: list):
    """
    Выполнение заданной пользователем команды с дополнительными аргументами
    :param command_name: название команды
    :param args: аргументы, которые будут переданы в функцию
    :return:
    """
    for key in commands.keys():
        if command_name in key:
            commands[key](*args)
        else:
            pass  # print("Command not found")


if __name__ == "__main__":

    # инициализация инструментов распознавания и ввода речи
    recognizer = speech_recognition.Recognizer()
    microphone = speech_recognition.Microphone()

    while True:
        # старт записи речи с последующим выводом распознанной речи
        # и удалением записанного в микрофон аудио
        voice_input = record_and_recognize_audio()
        os.remove("microphone-results.wav")
        print(voice_input)

        # отделение комманд от дополнительной информации (аргументов)
        voice_input = voice_input.split(" ")
        command = voice_input[0]
        command_options = [str(input_part) for input_part in voice_input[1:len(voice_input)]]
        execute_command_with_name(command, command_options)def search_for_video_on_youtube(*args: tuple):
    """
    Поиск видео на YouTube с автоматическим открытием ссылки на список результатов
    :param args: фраза поискового запроса
    """
    if not args[0]: return
    search_term = " ".join(args[0])
    url = "https://www.youtube.com/results?search_query=" + search_term
    webbrowser.get().open(url)

    # для мультиязычных голосовых ассистентов лучше создать 
    # отдельный класс, который будет брать перевод из JSON-файла
    play_voice_assistant_speech("Here is what I found for " + search_term + "on youtube")
    
  "Can you check if your microphone is on, please?": {
    "ru": "Пожалуйста, проверь, что микрофон включен",
    "en": "Can you check if your microphone is on, please?"
  },
  "What did you say again?": {
    "ru": "Пожалуйста, повтори",
    "en": "What did you say again?"
  },
}class Translation:
    """
    Получение вшитого в приложение перевода строк для 
    создания мультиязычного ассистента
    """
    with open("translations.json", "r", encoding="UTF-8") as file:
        translations = json.load(file)


    def get(self, text: str):
        """
        Получение перевода строки из файла на нужный язык (по его коду)
        :param text: текст, который требуется перевести
        :return: вшитый в приложение перевод текста
        """
        if text in self.translations:
            return self.translations[text][assistant.speech_language]
        else:
            # в случае отсутствия перевода происходит вывод сообщения 
            # об этом в логах и возврат исходного текста
            print(colored("Not translated phrase: {}".format(text), "red"))
            return textplay_voice_assistant_speech(translator.get(
    "Here is what I found for {} on Wikipedia").format(search_term))def prepare_corpus():
    """
    Подготовка модели для угадывания намерения пользователя
    """
    corpus = []
    target_vector = []
    for intent_name, intent_data in config["intents"].items():
        for example in intent_data["examples"]:
            corpus.append(example)
            target_vector.append(intent_name)

    training_vector = vectorizer.fit_transform(corpus)
    classifier_probability.fit(training_vector, target_vector)
    classifier.fit(training_vector, target_vector)


def get_intent(request):
    """
    Получение наиболее вероятного намерения в зависимости от запроса пользователя
    :param request: запрос пользователя
    :return: наиболее вероятное намерение
    """
    best_intent = classifier.predict(vectorizer.transform([request]))[0]

    index_of_best_intent = list(classifier_probability.classes_).index(best_intent)
    probabilities = classifier_probability.predict_proba(vectorizer.transform([request]))[0]

    best_intent_probability = probabilities[index_of_best_intent]

    # при добавлении новых намерений стоит уменьшать этот показатель
    if best_intent_probability > 0.57:
        return best_intentvectorizer = TfidfVectorizer(analyzer="char", ngram_range=(2, 3))
classifier_probability = LogisticRegression()
classifier = LinearSVC()
prepare_corpus()

while True:
    # старт записи речи с последующим выводом распознанной речи 
    # и удалением записанного в микрофон аудио
    voice_input = record_and_recognize_audio()

    if os.path.exists("microphone-results.wav"):
        os.remove("microphone-results.wav")

    print(colored(voice_input, "blue"))

    # отделение команд от дополнительной информации (аргументов)
    if voice_input:
        voice_input_parts = voice_input.split(" ")

        # если было сказано одно слово - выполняем команду сразу 
        # без дополнительных аргументов
        if len(voice_input_parts) == 1:
            intent = get_intent(voice_input)
            if intent:
                config["intents"][intent]["responses"]()
            else:
                config["failure_phrases"]()

        # в случае длинной фразы - выполняется поиск ключевой фразы 
        # и аргументов через каждое слово,
        # пока не будет найдено совпадение
        if len(voice_input_parts) > 1:
            for guess in range(len(voice_input_parts)):
                intent = get_intent((" ".join(voice_input_parts[0:guess])).strip())
                if intent:
                    command_options = [voice_input_parts[guess:len(voice_input_parts)]]
                    config["intents"][intent]["responses"](*command_options)
                    break
                if not intent and guess == len(voice_input_parts)-1:
                    config["failure_phrases"]()
temerlan123/temerlan123 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
