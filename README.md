An Innovative System for Monitoring Emotional Health to Identify Individuals at Risk of Psychological Disturbances
import numpy as np 
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
Data Reading
import json
​
with open('intents.json', 'r') as f:
    data = json.load(f)
​
df = pd.DataFrame(data['intents'])
df
tag	patterns	responses
0	greeting	[Hi, Hey, Is anyone there?, Hi there, Hello, H...	[Hello there. Tell me how are you feeling toda...
1	morning	[Good morning]	[Good morning. I hope you had a good night's s...
2	afternoon	[Good afternoon]	[Good afternoon. How is your day going?]
3	evening	[Good evening]	[Good evening. How has your day been?]
4	night	[Good night]	[Good night. Get some proper sleep, Good night...
...	...	...	...
75	fact-28	[What do I do if I'm worried about my mental h...	[The most important thing is to talk to someon...
76	fact-29	[How do I know if I'm unwell?]	[If your beliefs , thoughts , feelings or beha...
77	fact-30	[How can I maintain social connections? What i...	[A lot of people are alone right now, but we d...
78	fact-31	[What's the difference between anxiety and str...	[Stress and anxiety are often used interchange...
79	fact-32	[What's the difference between sadness and dep...	[Sadness is a normal reaction to a loss, disap...
80 rows × 3 columns

Each tag contains several questions(patterns) and several answers(responses); Now I want to seperate these patterns based on their tags and responses and finally convert them to a DataFrame.

    dic = {"tag":[], "patterns":[], "responses":[]}
    for i in range(len(df)):
        ptrns = df[df.index == i]['patterns'].values[0]
        rspns = df[df.index == i]['responses'].values[0]
        tag = df[df.index == i]['tag'].values[0]
        for j in range(len(ptrns)):
            dic['tag'].append(tag)
            dic['patterns'].append(ptrns[j])
            dic['responses'].append(rspns)
​
    df = pd.DataFrame.from_dict(dic)
    df
tag	patterns	responses
0	greeting	Hi	[Hello there. Tell me how are you feeling toda...
1	greeting	Hey	[Hello there. Tell me how are you feeling toda...
2	greeting	Is anyone there?	[Hello there. Tell me how are you feeling toda...
3	greeting	Hi there	[Hello there. Tell me how are you feeling toda...
4	greeting	Hello	[Hello there. Tell me how are you feeling toda...
...	...	...	...
227	fact-29	How do I know if I'm unwell?	[If your beliefs , thoughts , feelings or beha...
228	fact-30	How can I maintain social connections? What if...	[A lot of people are alone right now, but we d...
229	fact-31	What's the difference between anxiety and stress?	[Stress and anxiety are often used interchange...
230	fact-32	What's the difference between sadness and depr...	[Sadness is a normal reaction to a loss, disap...
231	fact-32	difference between sadness and depression	[Sadness is a normal reaction to a loss, disap...
232 rows × 3 columns

df['tag'].unique()
array(['greeting', 'morning', 'afternoon', 'evening', 'night', 'goodbye',
       'thanks', 'no-response', 'neutral-response', 'about', 'skill',
       'creation', 'name', 'help', 'sad', 'stressed', 'worthless',
       'depressed', 'happy', 'casual', 'anxious', 'not-talking', 'sleep',
       'scared', 'death', 'understand', 'done', 'suicide', 'hate-you',
       'hate-me', 'default', 'jokes', 'repeat', 'wrong', 'stupid',
       'location', 'something-else', 'friends', 'ask', 'problem',
       'no-approach', 'learn-more', 'user-agree', 'meditation',
       'user-meditation', 'pandora-useful', 'user-advice',
       'learn-mental-health', 'mental-health-fact', 'fact-1', 'fact-2',
       'fact-3', 'fact-5', 'fact-6', 'fact-7', 'fact-8', 'fact-9',
       'fact-10', 'fact-11', 'fact-12', 'fact-13', 'fact-14', 'fact-15',
       'fact-16', 'fact-17', 'fact-18', 'fact-19', 'fact-20', 'fact-21',
       'fact-22', 'fact-23', 'fact-24', 'fact-25', 'fact-26', 'fact-27',
       'fact-28', 'fact-29', 'fact-30', 'fact-31', 'fact-32'],
      dtype=object)
Data Preprocessing
Now I am going to apply some text preprocessing stuffs such as lowering, punctuation removing and then tokenize the patterns.

from tensorflow.keras.preprocessing.text import Tokenizer
​
tokenizer = Tokenizer(lower=True, split=' ')
tokenizer.fit_on_texts(df['patterns'])
tokenizer.get_config()
{'num_words': None,
 'filters': '!"#$%&()*+,-./:;<=>?@[\\]^_`{|}~\t\n',
 'lower': True,
 'split': ' ',
 'char_level': False,
 'oov_token': None,
 'document_count': 232,
 'word_counts': '{"hi": 2, "hey": 2, "is": 13, "anyone": 2, "there": 5, "hello": 1, "howdy": 1, "hola": 1, "bonjour": 1, "konnichiwa": 1, "guten": 1, "tag": 1, "ola": 1, "good": 5, "morning": 1, "afternoon": 1, "evening": 1, "night": 1, "bye": 3, "see": 2, "you": 38, "later": 1, "goodbye": 1, "au": 1, "revoir": 1, "sayonara": 1, "ok": 4, "then": 1, "fare": 1, "thee": 1, "well": 2, "thanks": 2, "thank": 3, "that\'s": 3, "helpful": 1, "for": 10, "the": 10, "help": 6, "than": 1, "very": 3, "much": 4, "nothing": 3, "who": 6, "are": 15, "what": 29, "tell": 6, "me": 19, "more": 6, "about": 20, "yourself": 3, "your": 3, "name": 4, "should": 3, "i": 95, "call": 1, "what\'s": 3, "can": 16, "do": 14, "created": 2, "how": 8, "were": 2, "made": 1, "my": 13, "am": 17, "go": 4, "by": 1, "could": 2, "give": 1, "a": 12, "hand": 1, "please": 2, "need": 6, "support": 3, "feeling": 1, "lonely": 4, "so": 10, "feel": 16, "down": 1, "sad": 2, "empty": 1, "don\'t": 11, "have": 9, "stressed": 4, "out": 3, "stuck": 1, "still": 1, "burned": 1, "worthless": 1, "no": 5, "one": 1, "likes": 1, "can\'t": 7, "anything": 2, "useless": 2, "makes": 1, "sense": 2, "anymore": 2, "take": 1, "it": 7, "depressed": 3, "think": 3, "i\'m": 9, "depression": 7, "great": 1, "today": 1, "happy": 2, "cheerful": 1, "fine": 2, "oh": 1, "okay": 1, "nice": 1, "whatever": 1, "k": 1, "yeah": 3, "yes": 2, "not": 5, "really": 2, "anxious": 2, "because": 4, "of": 8, "want": 9, "to": 24, "talk": 5, "just": 3, "stay": 1, "away": 4, "bring": 1, "myself": 7, "open": 1, "up": 2, "shut": 1, "insominia": 1, "suffering": 2, "from": 2, "insomnia": 1, "sleep": 3, "haven\'t": 2, "slept": 1, "last": 1, "days": 2, "seem": 1, "had": 1, "proper": 1, "past": 1, "few": 1, "scared": 2, "that": 8, "sounds": 3, "awful": 1, "this": 3, "way": 1, "mom": 1, "died": 3, "brother": 1, "dad": 1, "passed": 3, "sister": 1, "someone": 2, "in": 3, "family": 2, "friend": 1, "understand": 1, "you\'re": 5, "some": 4, "robot": 1, "would": 5, "know": 8, "possibly": 1, "going": 3, "through": 1, "nobody": 1, "understands": 1, "all": 4, "say": 2, "else": 4, "be": 2, "kill": 2, "i\'ve": 2, "thought": 1, "killing": 1, "die": 1, "commit": 1, "suicide": 1, "hate": 3, "like": 6, "trust": 1, "exams": 4, "friends": 2, "relationship": 1, "boyfriend": 1, "girlfriend": 1, "money": 1, "financial": 1, "problems": 5, "joke": 2, "another": 2, "already": 2, "told": 1, "mentioned": 1, "why": 2, "repeating": 1, "saying": 1, "doesn\'t": 1, "make": 1, "wrong": 2, "response": 1, "answer": 1, "stupid": 1, "crazy": 1, "dumb": 2, "where": 6, "live": 1, "location": 1, "something": 4, "let\'s": 1, "we": 1, "any": 2, "ask": 1, "probably": 2, "approaching": 1, "prepared": 1, "enough": 1, "guess": 2, "sure": 1, "learn": 6, "right": 3, "deserve": 1, "break": 1, "absolutely": 1, "hmmm": 1, "useful": 2, "did": 1, "said": 1, "and": 5, "alot": 1, "better": 2, "now": 1, "again": 1, "i\'ll": 1, "continue": 1, "practicing": 1, "meditation": 1, "focus": 1, "on": 2, "control": 1, "advice": 3, "mental": 25, "health": 19, "interested": 1, "learning": 1, "fact": 2, "define": 2, "important": 1, "importance": 1, "if": 6, "mentally": 1, "ill": 1, "therapist": 2, "does": 3, "therapy": 4, "mean": 1, "illness": 5, "affect": 1, "causes": 2, "warning": 1, "signs": 1, "people": 1, "with": 1, "recover": 1, "appears": 1, "symptoms": 1, "disorder": 1, "find": 4, "professional": 2, "or": 2, "child": 2, "treatment": 3, "options": 1, "available": 1, "become": 1, "involved": 1, "difference": 4, "between": 4, "professionals": 2, "get": 1, "before": 1, "starting": 1, "new": 1, "medication": 1, "types": 2, "different": 1, "group": 1, "prevent": 1, "cures": 1, "cure": 1, "worried": 1, "unwell": 1, "maintain": 1, "social": 1, "connections": 1, "anxiety": 1, "stress": 1, "sadness": 2}',
 'word_docs': '{"hi": 2, "hey": 2, "there": 5, "anyone": 2, "is": 13, "hello": 1, "howdy": 1, "hola": 1, "bonjour": 1, "konnichiwa": 1, "tag": 1, "guten": 1, "ola": 1, "morning": 1, "good": 5, "afternoon": 1, "evening": 1, "night": 1, "bye": 3, "you": 37, "see": 2, "later": 1, "goodbye": 1, "revoir": 1, "au": 1, "sayonara": 1, "ok": 4, "then": 1, "thee": 1, "well": 2, "fare": 1, "thanks": 2, "thank": 3, "that\'s": 3, "helpful": 1, "the": 10, "for": 10, "help": 6, "much": 4, "than": 1, "very": 3, "nothing": 3, "who": 6, "are": 15, "what": 29, "about": 20, "yourself": 3, "tell": 6, "more": 6, "me": 19, "name": 4, "your": 3, "i": 88, "should": 3, "call": 1, "what\'s": 3, "do": 12, "can": 16, "created": 2, "made": 1, "how": 8, "were": 2, "my": 13, "am": 17, "by": 1, "go": 4, "could": 2, "give": 1, "please": 2, "a": 12, "hand": 1, "need": 6, "support": 3, "feeling": 1, "lonely": 4, "so": 10, "down": 1, "feel": 16, "sad": 2, "empty": 1, "have": 9, "don\'t": 11, "out": 3, "stressed": 4, "stuck": 1, "still": 1, "burned": 1, "worthless": 1, "one": 1, "no": 5, "likes": 1, "anything": 2, "can\'t": 7, "useless": 2, "sense": 2, "anymore": 2, "makes": 1, "take": 1, "it": 7, "depressed": 3, "i\'m": 9, "think": 3, "depression": 7, "today": 1, "great": 1, "happy": 2, "cheerful": 1, "fine": 2, "oh": 1, "okay": 1, "nice": 1, "whatever": 1, "k": 1, "yeah": 3, "yes": 2, "not": 5, "really": 2, "anxious": 2, "because": 3, "of": 7, "talk": 5, "want": 9, "to": 23, "stay": 1, "just": 3, "away": 4, "myself": 7, "open": 1, "bring": 1, "up": 2, "shut": 1, "insominia": 1, "from": 2, "insomnia": 1, "suffering": 2, "sleep": 3, "days": 2, "haven\'t": 2, "last": 1, "slept": 1, "seem": 1, "past": 1, "had": 1, "proper": 1, "few": 1, "scared": 2, "sounds": 3, "awful": 1, "that": 8, "way": 1, "this": 3, "died": 3, "mom": 1, "brother": 1, "passed": 3, "dad": 1, "sister": 1, "in": 3, "family": 2, "someone": 2, "friend": 1, "understand": 1, "you\'re": 5, "would": 5, "know": 8, "some": 4, "robot": 1, "possibly": 1, "through": 1, "going": 3, "understands": 1, "nobody": 1, "all": 4, "say": 2, "else": 4, "be": 2, "kill": 2, "killing": 1, "i\'ve": 2, "thought": 1, "die": 1, "commit": 1, "suicide": 1, "hate": 3, "like": 6, "trust": 1, "exams": 4, "friends": 2, "relationship": 1, "boyfriend": 1, "girlfriend": 1, "money": 1, "problems": 5, "financial": 1, "joke": 2, "another": 2, "already": 2, "told": 1, "mentioned": 1, "repeating": 1, "why": 2, "saying": 1, "doesn\'t": 1, "make": 1, "wrong": 2, "response": 1, "answer": 1, "stupid": 1, "crazy": 1, "dumb": 2, "where": 6, "live": 1, "location": 1, "something": 4, "let\'s": 1, "we": 1, "any": 2, "ask": 1, "enough": 1, "approaching": 1, "prepared": 1, "probably": 2, "guess": 2, "sure": 1, "learn": 6, "break": 1, "deserve": 1, "right": 3, "absolutely": 1, "useful": 2, "hmmm": 1, "did": 1, "better": 2, "said": 1, "and": 5, "alot": 1, "now": 1, "again": 1, "i\'ll": 1, "meditation": 1, "on": 2, "control": 1, "continue": 1, "practicing": 1, "focus": 1, "advice": 3, "mental": 25, "health": 19, "interested": 1, "learning": 1, "fact": 2, "define": 2, "important": 1, "importance": 1, "if": 6, "mentally": 1, "ill": 1, "therapist": 2, "does": 3, "therapy": 4, "illness": 5, "mean": 1, "affect": 1, "causes": 2, "signs": 1, "warning": 1, "people": 1, "recover": 1, "with": 1, "appears": 1, "disorder": 1, "symptoms": 1, "find": 4, "professional": 2, "child": 2, "or": 2, "treatment": 3, "available": 1, "options": 1, "become": 1, "involved": 1, "between": 4, "professionals": 2, "difference": 4, "get": 1, "medication": 1, "new": 1, "before": 1, "starting": 1, "types": 2, "different": 1, "group": 1, "prevent": 1, "cures": 1, "cure": 1, "worried": 1, "unwell": 1, "social": 1, "connections": 1, "maintain": 1, "anxiety": 1, "stress": 1, "sadness": 2}',
 'index_docs': '{"95": 2, "96": 2, "41": 5, "97": 2, "14": 13, "150": 1, "151": 1, "152": 1, "153": 1, "154": 1, "156": 1, "155": 1, "157": 1, "158": 1, "42": 5, "159": 1, "160": 1, "161": 1, "68": 3, "2": 37, "98": 2, "162": 1, "163": 1, "165": 1, "164": 1, "166": 1, "51": 4, "167": 1, "169": 1, "99": 2, "168": 1, "100": 2, "69": 3, "70": 3, "170": 1, "19": 10, "18": 10, "32": 6, "52": 4, "171": 1, "71": 3, "72": 3, "33": 6, "12": 15, "3": 29, "6": 20, "73": 3, "34": 6, "35": 6, "7": 19, "53": 4, "74": 3, "1": 88, "75": 3, "172": 1, "76": 3, "13": 12, "10": 16, "101": 2, "173": 1, "24": 8, "102": 2, "15": 13, "9": 17, "174": 1, "54": 4, "103": 2, "175": 1, "104": 2, "16": 12, "176": 1, "36": 6, "77": 3, "177": 1, "55": 4, "20": 10, "178": 1, "11": 16, "105": 2, "179": 1, "21": 9, "17": 11, "78": 3, "56": 4, "180": 1, "181": 1, "182": 1, "183": 1, "184": 1, "43": 5, "185": 1, "106": 2, "28": 7, "107": 2, "108": 2, "109": 2, "186": 1, "187": 1, "29": 7, "79": 3, "22": 9, "80": 3, "30": 7, "189": 1, "188": 1, "110": 2, "190": 1, "111": 2, "191": 1, "192": 1, "193": 1, "194": 1, "195": 1, "81": 3, "112": 2, "44": 5, "113": 2, "114": 2, "57": 3, "25": 7, "45": 5, "23": 9, "5": 23, "196": 1, "82": 3, "58": 4, "31": 7, "198": 1, "197": 1, "115": 2, "199": 1, "200": 1, "117": 2, "201": 1, "116": 2, "83": 3, "119": 2, "118": 2, "203": 1, "202": 1, "204": 1, "207": 1, "205": 1, "206": 1, "208": 1, "120": 2, "84": 3, "209": 1, "26": 8, "210": 1, "85": 3, "86": 3, "211": 1, "212": 1, "87": 3, "213": 1, "214": 1, "88": 3, "122": 2, "121": 2, "215": 1, "216": 1, "46": 5, "47": 5, "27": 8, "59": 4, "217": 1, "218": 1, "219": 1, "89": 3, "221": 1, "220": 1, "60": 4, "123": 2, "61": 4, "124": 2, "125": 2, "223": 1, "126": 2, "222": 1, "224": 1, "225": 1, "226": 1, "90": 3, "37": 6, "227": 1, "62": 4, "127": 2, "228": 1, "229": 1, "230": 1, "231": 1, "48": 5, "232": 1, "128": 2, "129": 2, "130": 2, "233": 1, "234": 1, "235": 1, "131": 2, "236": 1, "237": 1, "238": 1, "132": 2, "239": 1, "240": 1, "241": 1, "242": 1, "133": 2, "38": 6, "243": 1, "244": 1, "63": 4, "245": 1, "246": 1, "134": 2, "247": 1, "250": 1, "248": 1, "249": 1, "135": 2, "136": 2, "251": 1, "39": 6, "253": 1, "252": 1, "91": 3, "254": 1, "137": 2, "255": 1, "256": 1, "138": 2, "257": 1, "49": 5, "258": 1, "259": 1, "260": 1, "261": 1, "264": 1, "139": 2, "266": 1, "262": 1, "263": 1, "265": 1, "92": 3, "4": 25, "8": 19, "267": 1, "268": 1, "140": 2, "141": 2, "269": 1, "270": 1, "40": 6, "271": 1, "272": 1, "142": 2, "93": 3, "64": 4, "50": 5, "273": 1, "274": 1, "143": 2, "276": 1, "275": 1, "277": 1, "279": 1, "278": 1, "280": 1, "282": 1, "281": 1, "65": 4, "144": 2, "146": 2, "145": 2, "94": 3, "284": 1, "283": 1, "285": 1, "286": 1, "67": 4, "147": 2, "66": 4, "287": 1, "291": 1, "290": 1, "288": 1, "289": 1, "148": 2, "292": 1, "293": 1, "294": 1, "295": 1, "296": 1, "297": 1, "298": 1, "300": 1, "301": 1, "299": 1, "302": 1, "303": 1, "149": 2}',
 'index_word': '{"1": "i", "2": "you", "3": "what", "4": "mental", "5": "to", "6": "about", "7": "me", "8": "health", "9": "am", "10": "can", "11": "feel", "12": "are", "13": "do", "14": "is", "15": "my", "16": "a", "17": "don\'t", "18": "for", "19": "the", "20": "so", "21": "have", "22": "i\'m", "23": "want", "24": "how", "25": "of", "26": "that", "27": "know", "28": "can\'t", "29": "it", "30": "depression", "31": "myself", "32": "help", "33": "who", "34": "tell", "35": "more", "36": "need", "37": "like", "38": "where", "39": "learn", "40": "if", "41": "there", "42": "good", "43": "no", "44": "not", "45": "talk", "46": "you\'re", "47": "would", "48": "problems", "49": "and", "50": "illness", "51": "ok", "52": "much", "53": "name", "54": "go", "55": "lonely", "56": "stressed", "57": "because", "58": "away", "59": "some", "60": "all", "61": "else", "62": "exams", "63": "something", "64": "therapy", "65": "find", "66": "difference", "67": "between", "68": "bye", "69": "thank", "70": "that\'s", "71": "very", "72": "nothing", "73": "yourself", "74": "your", "75": "should", "76": "what\'s", "77": "support", "78": "out", "79": "depressed", "80": "think", "81": "yeah", "82": "just", "83": "sleep", "84": "sounds", "85": "this", "86": "died", "87": "passed", "88": "in", "89": "going", "90": "hate", "91": "right", "92": "advice", "93": "does", "94": "treatment", "95": "hi", "96": "hey", "97": "anyone", "98": "see", "99": "well", "100": "thanks", "101": "created", "102": "were", "103": "could", "104": "please", "105": "sad", "106": "anything", "107": "useless", "108": "sense", "109": "anymore", "110": "happy", "111": "fine", "112": "yes", "113": "really", "114": "anxious", "115": "up", "116": "suffering", "117": "from", "118": "haven\'t", "119": "days", "120": "scared", "121": "someone", "122": "family", "123": "say", "124": "be", "125": "kill", "126": "i\'ve", "127": "friends", "128": "joke", "129": "another", "130": "already", "131": "why", "132": "wrong", "133": "dumb", "134": "any", "135": "probably", "136": "guess", "137": "useful", "138": "better", "139": "on", "140": "fact", "141": "define", "142": "therapist", "143": "causes", "144": "professional", "145": "or", "146": "child", "147": "professionals", "148": "types", "149": "sadness", "150": "hello", "151": "howdy", "152": "hola", "153": "bonjour", "154": "konnichiwa", "155": "guten", "156": "tag", "157": "ola", "158": "morning", "159": "afternoon", "160": "evening", "161": "night", "162": "later", "163": "goodbye", "164": "au", "165": "revoir", "166": "sayonara", "167": "then", "168": "fare", "169": "thee", "170": "helpful", "171": "than", "172": "call", "173": "made", "174": "by", "175": "give", "176": "hand", "177": "feeling", "178": "down", "179": "empty", "180": "stuck", "181": "still", "182": "burned", "183": "worthless", "184": "one", "185": "likes", "186": "makes", "187": "take", "188": "great", "189": "today", "190": "cheerful", "191": "oh", "192": "okay", "193": "nice", "194": "whatever", "195": "k", "196": "stay", "197": "bring", "198": "open", "199": "shut", "200": "insominia", "201": "insomnia", "202": "slept", "203": "last", "204": "seem", "205": "had", "206": "proper", "207": "past", "208": "few", "209": "awful", "210": "way", "211": "mom", "212": "brother", "213": "dad", "214": "sister", "215": "friend", "216": "understand", "217": "robot", "218": "possibly", "219": "through", "220": "nobody", "221": "understands", "222": "thought", "223": "killing", "224": "die", "225": "commit", "226": "suicide", "227": "trust", "228": "relationship", "229": "boyfriend", "230": "girlfriend", "231": "money", "232": "financial", "233": "told", "234": "mentioned", "235": "repeating", "236": "saying", "237": "doesn\'t", "238": "make", "239": "response", "240": "answer", "241": "stupid", "242": "crazy", "243": "live", "244": "location", "245": "let\'s", "246": "we", "247": "ask", "248": "approaching", "249": "prepared", "250": "enough", "251": "sure", "252": "deserve", "253": "break", "254": "absolutely", "255": "hmmm", "256": "did", "257": "said", "258": "alot", "259": "now", "260": "again", "261": "i\'ll", "262": "continue", "263": "practicing", "264": "meditation", "265": "focus", "266": "control", "267": "interested", "268": "learning", "269": "important", "270": "importance", "271": "mentally", "272": "ill", "273": "mean", "274": "affect", "275": "warning", "276": "signs", "277": "people", "278": "with", "279": "recover", "280": "appears", "281": "symptoms", "282": "disorder", "283": "options", "284": "available", "285": "become", "286": "involved", "287": "get", "288": "before", "289": "starting", "290": "new", "291": "medication", "292": "different", "293": "group", "294": "prevent", "295": "cures", "296": "cure", "297": "worried", "298": "unwell", "299": "maintain", "300": "social", "301": "connections", "302": "anxiety", "303": "stress"}',
 'word_index': '{"i": 1, "you": 2, "what": 3, "mental": 4, "to": 5, "about": 6, "me": 7, "health": 8, "am": 9, "can": 10, "feel": 11, "are": 12, "do": 13, "is": 14, "my": 15, "a": 16, "don\'t": 17, "for": 18, "the": 19, "so": 20, "have": 21, "i\'m": 22, "want": 23, "how": 24, "of": 25, "that": 26, "know": 27, "can\'t": 28, "it": 29, "depression": 30, "myself": 31, "help": 32, "who": 33, "tell": 34, "more": 35, "need": 36, "like": 37, "where": 38, "learn": 39, "if": 40, "there": 41, "good": 42, "no": 43, "not": 44, "talk": 45, "you\'re": 46, "would": 47, "problems": 48, "and": 49, "illness": 50, "ok": 51, "much": 52, "name": 53, "go": 54, "lonely": 55, "stressed": 56, "because": 57, "away": 58, "some": 59, "all": 60, "else": 61, "exams": 62, "something": 63, "therapy": 64, "find": 65, "difference": 66, "between": 67, "bye": 68, "thank": 69, "that\'s": 70, "very": 71, "nothing": 72, "yourself": 73, "your": 74, "should": 75, "what\'s": 76, "support": 77, "out": 78, "depressed": 79, "think": 80, "yeah": 81, "just": 82, "sleep": 83, "sounds": 84, "this": 85, "died": 86, "passed": 87, "in": 88, "going": 89, "hate": 90, "right": 91, "advice": 92, "does": 93, "treatment": 94, "hi": 95, "hey": 96, "anyone": 97, "see": 98, "well": 99, "thanks": 100, "created": 101, "were": 102, "could": 103, "please": 104, "sad": 105, "anything": 106, "useless": 107, "sense": 108, "anymore": 109, "happy": 110, "fine": 111, "yes": 112, "really": 113, "anxious": 114, "up": 115, "suffering": 116, "from": 117, "haven\'t": 118, "days": 119, "scared": 120, "someone": 121, "family": 122, "say": 123, "be": 124, "kill": 125, "i\'ve": 126, "friends": 127, "joke": 128, "another": 129, "already": 130, "why": 131, "wrong": 132, "dumb": 133, "any": 134, "probably": 135, "guess": 136, "useful": 137, "better": 138, "on": 139, "fact": 140, "define": 141, "therapist": 142, "causes": 143, "professional": 144, "or": 145, "child": 146, "professionals": 147, "types": 148, "sadness": 149, "hello": 150, "howdy": 151, "hola": 152, "bonjour": 153, "konnichiwa": 154, "guten": 155, "tag": 156, "ola": 157, "morning": 158, "afternoon": 159, "evening": 160, "night": 161, "later": 162, "goodbye": 163, "au": 164, "revoir": 165, "sayonara": 166, "then": 167, "fare": 168, "thee": 169, "helpful": 170, "than": 171, "call": 172, "made": 173, "by": 174, "give": 175, "hand": 176, "feeling": 177, "down": 178, "empty": 179, "stuck": 180, "still": 181, "burned": 182, "worthless": 183, "one": 184, "likes": 185, "makes": 186, "take": 187, "great": 188, "today": 189, "cheerful": 190, "oh": 191, "okay": 192, "nice": 193, "whatever": 194, "k": 195, "stay": 196, "bring": 197, "open": 198, "shut": 199, "insominia": 200, "insomnia": 201, "slept": 202, "last": 203, "seem": 204, "had": 205, "proper": 206, "past": 207, "few": 208, "awful": 209, "way": 210, "mom": 211, "brother": 212, "dad": 213, "sister": 214, "friend": 215, "understand": 216, "robot": 217, "possibly": 218, "through": 219, "nobody": 220, "understands": 221, "thought": 222, "killing": 223, "die": 224, "commit": 225, "suicide": 226, "trust": 227, "relationship": 228, "boyfriend": 229, "girlfriend": 230, "money": 231, "financial": 232, "told": 233, "mentioned": 234, "repeating": 235, "saying": 236, "doesn\'t": 237, "make": 238, "response": 239, "answer": 240, "stupid": 241, "crazy": 242, "live": 243, "location": 244, "let\'s": 245, "we": 246, "ask": 247, "approaching": 248, "prepared": 249, "enough": 250, "sure": 251, "deserve": 252, "break": 253, "absolutely": 254, "hmmm": 255, "did": 256, "said": 257, "alot": 258, "now": 259, "again": 260, "i\'ll": 261, "continue": 262, "practicing": 263, "meditation": 264, "focus": 265, "control": 266, "interested": 267, "learning": 268, "important": 269, "importance": 270, "mentally": 271, "ill": 272, "mean": 273, "affect": 274, "warning": 275, "signs": 276, "people": 277, "with": 278, "recover": 279, "appears": 280, "symptoms": 281, "disorder": 282, "options": 283, "available": 284, "become": 285, "involved": 286, "get": 287, "before": 288, "starting": 289, "new": 290, "medication": 291, "different": 292, "group": 293, "prevent": 294, "cures": 295, "cure": 296, "worried": 297, "unwell": 298, "maintain": 299, "social": 300, "connections": 301, "anxiety": 302, "stress": 303}'}
vacab_size = len(tokenizer.word_index)
print('number of unique words = ', vacab_size)
number of unique words =  303
​
from tensorflow.keras.preprocessing.sequence import pad_sequences
from sklearn.preprocessing import LabelEncoder
​
ptrn2seq = tokenizer.texts_to_sequences(df['patterns'])
X = pad_sequences(ptrn2seq, padding='post')
print('X shape = ', X.shape)
​
lbl_enc = LabelEncoder()
y = lbl_enc.fit_transform(df['tag'])
print('y shape = ', y.shape)
print('num of classes = ', len(np.unique(y)))
X shape =  (232, 18)
y shape =  (232,)
num of classes =  80
X
array([[ 95,   0,   0, ...,   0,   0,   0],
       [ 96,   0,   0, ...,   0,   0,   0],
       [ 14,  97,  41, ...,   0,   0,   0],
       ...,
       [ 76,  19,  66, ...,   0,   0,   0],
       [ 76,  19,  66, ...,   0,   0,   0],
       [ 66,  67, 149, ...,   0,   0,   0]])
y
array([44, 44, 44, 44, 44, 44, 44, 44, 44, 44, 44, 44, 55,  1, 10, 58, 43,
       43, 43, 43, 43, 43, 43, 43, 73, 73, 73, 73, 73, 60, 57,  0,  0,  0,
        0,  0,  0,  0,  0, 67,  5,  5,  5, 56, 56, 56, 48, 48, 48, 48, 48,
       48, 48, 65, 65, 65, 65, 65, 65, 65, 65, 70, 70, 70, 70, 70, 78, 78,
       78, 78, 78,  8,  8,  8,  8, 45, 45, 45, 45, 45, 45, 45,  4,  4,  4,
        4,  4,  4,  4,  4,  4,  4,  4,  2,  2, 61, 61, 61, 61, 68, 68, 68,
       68, 68, 68, 66, 66, 66, 66,  6,  6,  6,  6,  6,  6, 74, 74, 74, 74,
       74, 74,  9,  9,  9,  9,  9, 72, 72, 72, 72, 72, 47, 47, 47, 46, 46,
       46,  7,  7,  7,  7,  7,  7,  7,  7, 49, 49, 64, 64, 64, 79, 79, 79,
       79, 71, 71, 71, 71, 52, 52, 52, 69, 69, 69, 69, 42,  3, 63, 63, 59,
       59, 59, 51, 51, 51, 76, 76, 53, 53, 77, 77, 62, 75, 75, 75, 50, 50,
       50, 54, 54, 11, 11, 22, 22, 33, 33, 37, 37, 37, 37, 38, 38, 39, 39,
       39, 40, 41, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 23, 24, 25, 26,
       27, 28, 29, 29, 30, 31, 32, 34, 35, 36, 36])
Build and Train Model
import tensorflow
from tensorflow import keras
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Input, Embedding, LSTM, LayerNormalization, Dense, Dropout
from tensorflow.keras.utils import plot_model
​
model = Sequential()
model.add(Input(shape=(X.shape[1])))
model.add(Embedding(input_dim=vacab_size+1, output_dim=100, mask_zero=True))
model.add(LSTM(32, return_sequences=True))
model.add(LayerNormalization())
model.add(LSTM(32, return_sequences=True))
model.add(LayerNormalization())
model.add(LSTM(32))
model.add(LayerNormalization())
model.add(Dense(128, activation="relu"))
model.add(LayerNormalization())
model.add(Dropout(0.2))
model.add(Dense(128, activation="relu"))
model.add(LayerNormalization())
model.add(Dropout(0.2))
model.add(Dense(len(np.unique(y)), activation="softmax"))
model.compile(optimizer='adam', loss="sparse_categorical_crossentropy", metrics=['accuracy'])
​
model.summary()
plot_model(model, show_shapes=True)
WARNING:tensorflow:From C:\Users\Manvitha\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.7_qbz5n2kfra8p0\LocalCache\local-packages\Python37\site-packages\tensorflow\python\keras\initializers.py:119: calling RandomUniform.__init__ (from tensorflow.python.ops.init_ops) with dtype is deprecated and will be removed in a future version.
Instructions for updating:
Call initializer instance with the dtype argument instead of passing it to the constructor
WARNING:tensorflow:From C:\Users\Manvitha\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.7_qbz5n2kfra8p0\LocalCache\local-packages\Python37\site-packages\tensorflow\python\ops\init_ops.py:1251: calling VarianceScaling.__init__ (from tensorflow.python.ops.init_ops) with dtype is deprecated and will be removed in a future version.
Instructions for updating:
Call initializer instance with the dtype argument instead of passing it to the constructor
WARNING:tensorflow:From C:\Users\Manvitha\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.7_qbz5n2kfra8p0\LocalCache\local-packages\Python37\site-packages\tensorflow\python\keras\backend.py:3794: add_dispatch_support.<locals>.wrapper (from tensorflow.python.ops.array_ops) is deprecated and will be removed in a future version.
Instructions for updating:
Use tf.where in 2.0, which has the same broadcast rule as np.where
Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
embedding (Embedding)        (None, 18, 100)           30400     
_________________________________________________________________
lstm (LSTM)                  (None, 18, 32)            17024     
_________________________________________________________________
layer_normalization (LayerNo (None, 18, 32)            64        
_________________________________________________________________
lstm_1 (LSTM)                (None, 18, 32)            8320      
_________________________________________________________________
layer_normalization_1 (Layer (None, 18, 32)            64        
_________________________________________________________________
lstm_2 (LSTM)                (None, 32)                8320      
_________________________________________________________________
layer_normalization_2 (Layer (None, 32)                64        
_________________________________________________________________
dense (Dense)                (None, 128)               4224      
_________________________________________________________________
layer_normalization_3 (Layer (None, 128)               256       
_________________________________________________________________
dropout (Dropout)            (None, 128)               0         
_________________________________________________________________
dense_1 (Dense)              (None, 128)               16512     
_________________________________________________________________
layer_normalization_4 (Layer (None, 128)               256       
_________________________________________________________________
dropout_1 (Dropout)          (None, 128)               0         
_________________________________________________________________
dense_2 (Dense)              (None, 80)                10320     
=================================================================
Total params: 95,824
Trainable params: 95,824
Non-trainable params: 0
_________________________________________________________________
Failed to import pydot. You must install pydot and graphviz for `pydotprint` to work.
model_history = model.fit(x=X,
                          y=y,
                          batch_size=10,
                          callbacks=[tensorflow.keras.callbacks.EarlyStopping(monitor='accuracy', patience=3)],
                          epochs=50)
Epoch 1/50
230/232 [============================>.] - ETA: 0s - loss: 4.9608 - acc: 0.0174WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 5s 22ms/sample - loss: 4.9493 - acc: 0.0172
Epoch 2/50
230/232 [============================>.] - ETA: 0s - loss: 3.7284 - acc: 0.1087WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 3.7348 - acc: 0.1121
Epoch 3/50
230/232 [============================>.] - ETA: 0s - loss: 2.8943 - acc: 0.3000WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 2.8985 - acc: 0.3017
Epoch 4/50
230/232 [============================>.] - ETA: 0s - loss: 2.5510 - acc: 0.3391WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 2.5518 - acc: 0.3362
Epoch 5/50
230/232 [============================>.] - ETA: 0s - loss: 2.0890 - acc: 0.4783WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 2.0785 - acc: 0.4828
Epoch 6/50
230/232 [============================>.] - ETA: 0s - loss: 1.7401 - acc: 0.5739WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 1.7373 - acc: 0.5776
Epoch 7/50
230/232 [============================>.] - ETA: 0s - loss: 1.3722 - acc: 0.6913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 1.3727 - acc: 0.6940
Epoch 8/50
230/232 [============================>.] - ETA: 0s - loss: 1.1241 - acc: 0.7522- ETA: 1s - loss: 1.WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 1.1187 - acc: 0.7543
Epoch 9/50
230/232 [============================>.] - ETA: 0s - loss: 0.9467 - acc: 0.8217WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.9495 - acc: 0.8190
Epoch 10/50
230/232 [============================>.] - ETA: 0s - loss: 0.8318 - acc: 0.8478WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.8305 - acc: 0.8491
Epoch 11/50
230/232 [============================>.] - ETA: 0s - loss: 0.7107 - acc: 0.8739WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.7071 - acc: 0.8750
Epoch 12/50
230/232 [============================>.] - ETA: 0s - loss: 0.5697 - acc: 0.9174WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.5748 - acc: 0.9138
Epoch 13/50
230/232 [============================>.] - ETA: 0s - loss: 0.4566 - acc: 0.9217WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.4689 - acc: 0.9181
Epoch 14/50
230/232 [============================>.] - ETA: 0s - loss: 0.4581 - acc: 0.9261WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.4546 - acc: 0.9267
Epoch 15/50
230/232 [============================>.] - ETA: 0s - loss: 0.3114 - acc: 0.9609WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.3093 - acc: 0.9612
Epoch 16/50
230/232 [============================>.] - ETA: 0s - loss: 0.3188 - acc: 0.9478WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.3173 - acc: 0.9483
Epoch 17/50
230/232 [============================>.] - ETA: 0s - loss: 0.2368 - acc: 0.9783WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 1s 6ms/sample - loss: 0.2352 - acc: 0.9784
Epoch 18/50
230/232 [============================>.] - ETA: 0s - loss: 0.2133 - acc: 0.9739WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 1s 6ms/sample - loss: 0.2124 - acc: 0.9741
Epoch 19/50
230/232 [============================>.] - ETA: 0s - loss: 0.1930 - acc: 0.9783WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.1939 - acc: 0.9784
Epoch 20/50
230/232 [============================>.] - ETA: 0s - loss: 0.1804 - acc: 0.9870WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.1838 - acc: 0.9871
Epoch 21/50
230/232 [============================>.] - ETA: 0s - loss: 0.1789 - acc: 0.9652WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.1790 - acc: 0.9655
Epoch 22/50
230/232 [============================>.] - ETA: 0s - loss: 0.1401 - acc: 0.9783WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.1390 - acc: 0.9784
Epoch 23/50
230/232 [============================>.] - ETA: 0s - loss: 0.1107 - acc: 0.9826WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.1106 - acc: 0.9828
Epoch 24/50
230/232 [============================>.] - ETA: 0s - loss: 0.1052 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.1053 - acc: 0.9914
Epoch 25/50
230/232 [============================>.] - ETA: 0s - loss: 0.0775 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0784 - acc: 0.9957
Epoch 26/50
230/232 [============================>.] - ETA: 0s - loss: 0.0855 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0852 - acc: 0.9957
Epoch 27/50
230/232 [============================>.] - ETA: 0s - loss: 0.0903 - acc: 0.9826WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0896 - acc: 0.9828
Epoch 28/50
230/232 [============================>.] - ETA: 0s - loss: 0.0658 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0660 - acc: 0.9957
Epoch 29/50
230/232 [============================>.] - ETA: 0s - loss: 0.0641 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0650 - acc: 0.9957
Epoch 30/50
230/232 [============================>.] - ETA: 0s - loss: 0.0712 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0709 - acc: 0.9914
Epoch 31/50
230/232 [============================>.] - ETA: 0s - loss: 0.0767 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0765 - acc: 0.9914
Epoch 32/50
230/232 [============================>.] - ETA: 0s - loss: 0.0504 - acc: 1.0000WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0506 - acc: 1.0000
Epoch 33/50
230/232 [============================>.] - ETA: 0s - loss: 0.0616 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0621 - acc: 0.9914
Epoch 34/50
230/232 [============================>.] - ETA: 0s - loss: 0.0529 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0526 - acc: 0.9914
Epoch 35/50
230/232 [============================>.] - ETA: 0s - loss: 0.0478 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0475 - acc: 0.9957
Epoch 36/50
230/232 [============================>.] - ETA: 0s - loss: 0.0427 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0425 - acc: 0.9957
Epoch 37/50
230/232 [============================>.] - ETA: 0s - loss: 0.0410 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0411 - acc: 0.9957
Epoch 38/50
230/232 [============================>.] - ETA: 0s - loss: 0.0408 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0405 - acc: 0.9957
Epoch 39/50
230/232 [============================>.] - ETA: 0s - loss: 0.0415 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0419 - acc: 0.9957
Epoch 40/50
230/232 [============================>.] - ETA: 0s - loss: 0.0393 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0394 - acc: 0.9957
Epoch 41/50
230/232 [============================>.] - ETA: 0s - loss: 0.0312 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0310 - acc: 0.9957
Epoch 42/50
230/232 [============================>.] - ETA: 0s - loss: 0.0344 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0341 - acc: 0.9957
Epoch 43/50
230/232 [============================>.] - ETA: 0s - loss: 0.0403 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0403 - acc: 0.9914
Epoch 44/50
230/232 [============================>.] - ETA: 0s - loss: 0.0319 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0322 - acc: 0.9914
Epoch 45/50
230/232 [============================>.] - ETA: 0s - loss: 0.0228 - acc: 1.0000WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0228 - acc: 1.0000
Epoch 46/50
230/232 [============================>.] - ETA: 0s - loss: 0.0276 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0282 - acc: 0.9914
Epoch 47/50
230/232 [============================>.] - ETA: 0s - loss: 0.0364 - acc: 0.9913WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 8ms/sample - loss: 0.0362 - acc: 0.9914
Epoch 48/50
230/232 [============================>.] - ETA: 0s - loss: 0.0281 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0279 - acc: 0.9957
Epoch 49/50
230/232 [============================>.] - ETA: 0s - loss: 0.0194 - acc: 1.0000WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 7ms/sample - loss: 0.0193 - acc: 1.0000
Epoch 50/50
230/232 [============================>.] - ETA: 0s - loss: 0.0269 - acc: 0.9957WARNING:tensorflow:Early stopping conditioned on metric `accuracy` which is not available. Available metrics are: loss,acc
232/232 [==============================] - 2s 6ms/sample - loss: 0.0266 - acc: 0.9957
Model Testing
First we should apply some text preprocessing on the pattern that is passed to the function. Next we convert the text to vector of numbers and give it to model for prediction its tag; Finally based on the tag, we choose a answer(response) randomly and return it.

import re
import random
​
def generate_answer(pattern): 
    text = []
    txt = re.sub('[^a-zA-Z\']', ' ', pattern)
    txt = txt.lower()
    txt = txt.split()
    txt = " ".join(txt)
    text.append(txt)
        
    x_test = tokenizer.texts_to_sequences(text)
    x_test = np.array(x_test).squeeze()
    x_test = pad_sequences([x_test], padding='post', maxlen=X.shape[1])
    y_pred = model.predict(x_test)
    y_pred = y_pred.argmax()
    tag = lbl_enc.inverse_transform([y_pred])[0]
    responses = df[df['tag'] == tag]['responses'].values[0]
   # print(responses)
    print("you: {}".format(pattern))
    print("model: {}".format(random.choice(responses)))
generate_answer('Hi! How are you?')
you: Hi! How are you?
model: Hello there. Glad to see you're back. What's going on in your world right now?
generate_answer('Maybe I just didn\'t want to be born :)')
you: Maybe I just didn't want to be born :)
model: You can talk to me without fear of judgement.
generate_answer('help me:')
you: help me:
model: I'm sorry to hear that. I'm doing my best to help
generate_answer(':')
you: :
model: Sorry, I didn't understand you.
generate_answer('Good morning')
you: Good morning
model: Good morning. I hope you had a good night's sleep. How are you feeling today? 
def chatbot():
    print("Chatbot: Hi! I'm your friendly chatbot. How can I assist you today?")
    
    while True:
        user_input = input("You: ")
        if user_input.lower() in ['quit', 'exit', 'q', 'bye']:
            print("Chatbot: Goodbye!")
            break
​
        generate_answer(user_input)
​
if __name__ == "__main__":
    chatbot()
Chatbot: Hi! I'm your friendly chatbot. How can I assist you today?
You: Good morning
you: Good morning
model: Good morning. I hope you had a good night's sleep. How are you feeling today? 
You: :
you: :
model: Please go on.
You: help me:
you: help me:
model: I'm trying my best to help you. So please talk to me
