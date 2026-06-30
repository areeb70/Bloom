import 'package:flutter/material.dart';
import 'dart:math';
import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:intl/intl.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await GlobalSettings.load();
  runApp(const BloomApp());
}

// --- GLOBAL STATE MANAGER ---
class GlobalSettings {
  static ValueNotifier<Color> themeColor = ValueNotifier(Colors.teal);
  static ValueNotifier<String> language = ValueNotifier('en');
  static ValueNotifier<ThemeMode> themeMode = ValueNotifier(ThemeMode.light);

  static Future<void> load() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    themeColor.value = Color(prefs.getInt('themeColor') ?? Colors.teal.value);
    themeMode.value = ThemeMode.values[prefs.getInt('themeMode') ?? 0];
    language.value = prefs.getString('language') ?? 'en';
  }

  static Future<void> saveTheme(Color color) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setInt('themeColor', color.value);
    themeColor.value = color;
  }

  static Future<void> saveLanguage(String lang) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setString('language', lang);
    language.value = lang;
  }

  static Future<void> saveThemeMode(ThemeMode mode) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setInt('themeMode', mode.index);
    themeMode.value = mode;
  }
}

class BloomApp extends StatelessWidget {
  const BloomApp({super.key});

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder(
      valueListenable: GlobalSettings.themeColor,
      builder: (context, color, _) {
        return ValueListenableBuilder(
          valueListenable: GlobalSettings.themeMode, // Added this listener
          builder: (context, mode, _) {
            return ValueListenableBuilder(
              valueListenable: GlobalSettings.language,
              builder: (context, lang, _) {
                return MaterialApp(
                  debugShowCheckedModeBanner: false,
                  title: 'Bloom',
                  theme: ThemeData(
                    colorScheme: ColorScheme.fromSeed(
                      seedColor: color,
                      brightness: Brightness.light,
                    ),
                    useMaterial3: true,
                    fontFamily: 'Georgia',
                  ),
                  darkTheme: ThemeData(
                    colorScheme: ColorScheme.fromSeed(
                      seedColor: color,
                      brightness: Brightness.dark,
                    ),
                    useMaterial3: true,
                    fontFamily: 'Georgia',
                  ),
                  themeMode: mode, // This tells Flutter which theme to use
                  home: SplashScreen(
                    onUserFound: (name) => LevelMapScreen(userName: name),
                    onNoUser: () => const WelcomeScreen(),
                  ),
                );
              },
            );
          },
        );
      },
    );
  }
}

// --- TRANSLATIONS ---
class AppTexts {
  static final Map<String, Map<String, String>> translations = {
    'en': {
      'welcome': 'Welcome to Bloom',
      'subtitle': 'A safe space to grow your confidence.',
      'start': 'Start My Journey',
      'guest': 'Continue as Guest',
      'progress': 'Your Growth Progress',
      'choose_level': 'Choose your growth stage:',
      'hello': 'Hello',
      'profile': 'My Profile',
      'history': 'My Growth Journey',
      'reflect': 'Reflect on your growth',
      'anxiety_q': 'How anxious did you feel? (1-10)',
      'what_happened': 'What actually happened?',
      'finish': 'Finish Reflection',
      'streak': 'Current Streak',
      'best': 'Best Streak',
    },
    'es': {
      'welcome': 'Bienvenido a Bloom',
      'subtitle': 'Un espacio seguro para crecer tu confianza.',
      'start': 'Comenzar mi viaje',
      'guest': 'Continuar como invitado',
      'progress': 'Tu progreso de crecimiento',
      'choose_level': 'Elige tu etapa de crecimiento:',
      'hello': 'Hola',
      'profile': 'Mi Perfil',
      'history': 'Mi Viaje de Crecimiento',
      'reflect': 'Reflexiona sobre tu crecimiento',
      'anxiety_q': '¿Qué tan ansioso te sentiste? (1-10)',
      'what_happened': '¿Qué pasó realmente?',
      'finish': 'Terminar Reflexión',
      'streak': 'Racha Actual',
      'best': 'Mejor Racha',
    },
    'fr': {
      'welcome': 'Bienvenue chez Bloom',
      'subtitle': 'Un espace sûr pour développer votre confiance.',
      'start': 'Commencer mon voyage',
      'guest': 'Continuer en tant qu\'invité',
      'progress': 'Votre progrès de croissance',
      'choose_level': 'Choisissez votre étape de croissance :',
      'hello': 'Bonjour',
      'profile': 'Mon Profil',
      'history': 'Mon Voyage de Croissance',
      'reflect': 'Réfléchissez à votre croissance',
      'anxiety_q': 'À quel point vous sentiez-vous anxieux ? (1-10)',
      'what_happened': 'Que s\'est-il passé réellement ?',
      'finish': 'Terminer la réflexion',
      'streak': 'Série actuelle',
      'best': 'Meilleure série',
    },
    'hi': {
      'welcome': 'ब्लूम में आपका स्वागत है',
      'subtitle': 'अपने आत्मविश्वास को बढ़ाने के लिए एक सुरक्षित स्थान।',
      'start': 'मेरी यात्रा शुरू करें',
      'guest': 'अतिथि के रूप में जारी रखें',
      'progress': 'आपकी विकास प्रगति',
      'choose_level': 'अपना विकास चरण चुनें:',
      'hello': 'नमस्ते',
      'profile': 'मेरी प्रोफाइल',
      'history': 'मेरी विकास यात्रा',
      'reflect': 'अपनी वृद्धि पर विचार करें',
      'anxiety_q': 'आप कितना चिंतित महसूस कर रहे थे? (1-10)',
      'what_happened': 'असल में क्या हुआ?',
      'finish': 'चिंतन समाप्त करें',
      'streak': 'वर्तमान सिलसिला',
      'best': 'सर्वश्रेष्ठ सिलसिला',
    },
  };

  static String get(String key, String lang) {
    return translations[lang]?[key] ?? translations['en']![key]!;
  }
}

// --- TASK TRANSLATIONS ---
class TaskLibrary {
  static final Map<String, Map<String, List<Map<String, String>>>>
  translations = {
    'en': {
      "Seedling": [
        {
          "id": "S1",
          "title": "The First Step",
          "desc": "Make eye contact and smile at one person today.",
        },
        {
          "id": "S2",
          "title": "A Simple Hello",
          "desc": "Say 'Good morning' or 'Hello' to a neighbor.",
        },
        {
          "id": "S3",
          "title": "The Thank You",
          "desc": "Say 'Thank you' clearly to a shopkeeper.",
        },
        {
          "id": "S4",
          "title": "The Observation",
          "desc": "Notice something positive about a stranger and smile.",
        },
        {
          "id": "S5",
          "title": "The Quiet Wave",
          "desc": "Wave at someone you recognize from a distance.",
        },
        {
          "id": "S6",
          "title": "Door Hold",
          "desc": "Hold the door open for someone behind you.",
        },
        {
          "id": "S7",
          "title": "The Nod",
          "desc": "Give a friendly nod to a colleague as you pass them.",
        },
        {
          "id": "S8",
          "title": "The Mirror",
          "desc": "Practice your 'confident smile' in the mirror for 1 minute.",
        },
        {
          "id": "S9",
          "title": "The Brief Glance",
          "desc": "Look at someone for 2 seconds, then smile and look away.",
        },
        {
          "id": "S10",
          "title": "The Quiet Praise",
          "desc": "Write a nice comment on someone's social media post.",
        },
        {
          "id": "S11",
          "title": "The Space Share",
          "desc":
              "Sit next to someone in a public area without looking away immediately.",
        },
        {
          "id": "S12",
          "title": "The Simple Acknowledgement",
          "desc": "Say 'Excuse me' politely when passing someone in a hallway.",
        },
        {
          "id": "S13",
          "title": "The Warm Greeting",
          "desc": "Say 'Hi' to a delivery driver or courier.",
        },
        {
          "id": "S14",
          "title": "The Small Wave",
          "desc": "Wave to a child or a pet (with owner's permission).",
        },
        {
          "id": "S15",
          "title": "The Soft Smile",
          "desc": "Smile at three different people today.",
        },
        {
          "id": "S16",
          "title": "The Eye-Contact Challenge",
          "desc":
              "Maintain eye contact with a cashier until they look away first.",
        },
        {
          "id": "S17",
          "title": "The Gentle Breath",
          "desc": "Take 3 deep breaths before entering a social space today.",
        },
        {
          "id": "S18",
          "title": "The Presence",
          "desc":
              "Stand in a crowded area for 5 minutes without looking at your phone.",
        },
        {
          "id": "S19",
          "title": "The Casual Nod",
          "desc": "Nod to a stranger who makes eye contact with you.",
        },
        {
          "id": "S20",
          "title": "The Soft Voice",
          "desc": "Say 'Have a nice day' to someone as you leave a store.",
        },
      ],
      "Sprout": [
        {
          "id": "SP1",
          "title": "The Compliment",
          "desc": "Give a genuine compliment to a colleague or classmate.",
        },
        {
          "id": "SP2",
          "title": "The Question",
          "desc": "Ask a stranger for the time or directions.",
        },
        {
          "id": "SP3",
          "title": "Small Talk",
          "desc":
              "Ask someone 'How is your day going?' and listen to the answer.",
        },
        {
          "id": "SP4",
          "title": "The Request",
          "desc": "Ask a store employee for help finding a specific item.",
        },
        {
          "id": "SP5",
          "title": "The Order",
          "desc": "Order a drink or food and ask the staff how they are doing.",
        },
        {
          "id": "SP6",
          "title": "The Greeting",
          "desc": "Introduce yourself to someone new in your area.",
        },
        {
          "id": "SP7",
          "title": "The Weather Talk",
          "desc": "Mention the weather to someone while waiting in a line.",
        },
        {
          "id": "SP8",
          "title": "The Simple Inquiry",
          "desc": "Ask a coworker 'What did you do over the weekend?'",
        },
        {
          "id": "SP9",
          "title": "The Help Offer",
          "desc":
              "Ask someone 'Do you need help with that?' if they look struggling.",
        },
        {
          "id": "SP10",
          "title": "The Opinion",
          "desc":
              "Ask a friend 'What do you think of this?' about a small object.",
        },
        {
          "id": "SP11",
          "title": "The Confirmation",
          "desc":
              "Confirm a detail with a stranger (e.g., 'Is this the right line?').",
        },
        {
          "id": "SP12",
          "title": "The Shared Space",
          "desc":
              "Make a small comment about the environment (e.g., 'It's really crowded').",
        },
        {
          "id": "SP13",
          "title": "The Small Favor",
          "desc":
              "Ask someone to pass you something (like a napkin) at a table.",
        },
        {
          "id": "SP14",
          "title": "The Warm Feedback",
          "desc": "Tell a waiter that the food was great before leaving.",
        },
        {
          "id": "SP15",
          "title": "The Casual Check-in",
          "desc":
              "Send a 'How are you?' text to someone you haven't spoken to in a month.",
        },
        {
          "id": "SP16",
          "title": "The Open Question",
          "desc":
              "Ask someone 'Where is your favorite place to visit in this city?'",
        },
        {
          "id": "SP17",
          "title": "The Smallest Risk",
          "desc": "Ask a stranger if they know where the nearest restroom is.",
        },
        {
          "id": "SP18",
          "title": "The Item Praise",
          "desc": "Tell someone you like their shoes/bag/accessory.",
        },
        {
          "id": "SP19",
          "title": "The Polite Pause",
          "desc":
              "Wait for someone to finish speaking entirely before responding to them.",
        },
        {
          "id": "SP20",
          "title": "The Friendly Wave",
          "desc":
              "Wave and say 'Bye' to someone you just had a short interaction with.",
        },
      ],
      "Leaf": [
        {
          "id": "L1",
          "title": "Opinion Seeker",
          "desc": "Ask someone for their opinion on a book, movie, or song.",
        },
        {
          "id": "L2",
          "title": "The Detail",
          "desc":
              "Ask a follow-up question after someone tells you something about themselves.",
        },
        {
          "id": "L3",
          "title": "The Recommendation",
          "desc":
              "Ask a stranger for a recommendation for a good place to eat nearby.",
        },
        {
          "id": "L4",
          "title": "The Connection",
          "desc":
              "Find a common interest with someone and talk about it for 2 minutes.",
        },
        {
          "id": "L5",
          "title": "The Helpful Hand",
          "desc":
              "Offer to help someone with a small task (like carrying a bag).",
        },
        {
          "id": "L6",
          "title": "The Social Observation",
          "desc":
              "Start a conversation based on something happening around you both.",
        },
        {
          "id": "L7",
          "title": "The Open Ended Question",
          "desc": "Ask someone 'How did you get into this line of work?'",
        },
        {
          "id": "L8",
          "title": "The Active Listener",
          "desc":
              "Listen to someone for 3 minutes without interrupting, then summarize what they said.",
        },
        {
          "id": "L9",
          "title": "The Shared Laugh",
          "desc": "Tell a short, funny story or a joke to a small group.",
        },
        {
          "id": "L10",
          "title": "The Curiosity",
          "desc":
              "Ask someone where they are from and what they like about that place.",
        },
        {
          "id": "L11",
          "title": "The Sincere Interest",
          "desc": "Ask a colleague about their hobbies outside of work.",
        },
        {
          "id": "L12",
          "title": "The Soft Advice",
          "desc": "Give someone a helpful tip on something you are good at.",
        },
        {
          "id": "L13",
          "title": "The Group Nod",
          "desc": "Agree with someone's point in a small group discussion.",
        },
        {
          "id": "L14",
          "title": "The Casual Invitation",
          "desc": "Ask someone 'Would you like to join us for lunch?'",
        },
        {
          "id": "L15",
          "title": "The Honest Reflection",
          "desc":
              "Tell someone 'I really appreciated it when you did X' and explain why.",
        },
        {
          "id": "L16",
          "title": "The Curiosity Gap",
          "desc":
              "Ask someone 'I've always wondered, how does X actually work?'",
        },
        {
          "id": "L17",
          "title": "The Small Group Lead",
          "desc":
              "Ask a question that requires 2 or 3 people in a group to answer.",
        },
        {
          "id": "L18",
          "title": "The Genuine Compliment",
          "desc":
              "Compliment someone on a personality trait (e.g., 'You're a great listener').",
        },
        {
          "id": "L19",
          "title": "The Shared Experience",
          "desc":
              "Say 'I've been in that situation too' during a conversation.",
        },
        {
          "id": "L20",
          "title": "The Meaningful Pause",
          "desc":
              "Allow a silence to happen in a conversation without rushing to fill it.",
        },
      ],
      "Stem": [
        {
          "id": "ST1",
          "title": "The Brave Start",
          "desc": "Start a conversation with someone you don't know well.",
        },
        {
          "id": "ST2",
          "title": "The Honest Share",
          "desc": "Share a small personal story or opinion in a group setting.",
        },
        {
          "id": "ST3",
          "title": "The Debate",
          "desc": "Politely disagree with someone's opinion and explain why.",
        },
        {
          "id": "ST4",
          "title": "The Group Entry",
          "desc":
              "Join a group conversation and contribute a thoughtful sentence.",
        },
        {
          "id": "ST4",
          "title": "The Topic Lead",
          "desc": "Bring up a new topic of conversation in a social group.",
        },
        {
          "id": "ST6",
          "title": "The Public Question",
          "desc": "Ask a question in a public meeting or a classroom setting.",
        },
        {
          "id": "ST7",
          "title": "The Bold Request",
          "desc":
              "Ask a stranger if you can sit next to them at a cafe or park.",
        },
        {
          "id": "ST8",
          "title": "The Conversation Bridge",
          "desc":
              "Introduce two people who don't know each other and find a commonality.",
        },
        {
          "id": "ST9",
          "title": "The Assertive Need",
          "desc":
              "Politely ask someone to move or stop doing something that bothers you.",
        },
        {
          "id": "ST10",
          "title": "The Storyteller",
          "desc":
              "Take the lead in telling a story to a group of 3 or more people.",
        },
        {
          "id": "ST11",
          "title": "The Open Challenge",
          "desc":
              "Challenge a common opinion in a group in a friendly, respectful way.",
        },
        {
          "id": "ST12",
          "title": "The Social Initiative",
          "desc":
              "Be the first person to say 'Hello' to everyone when entering a room.",
        },
        {
          "id": "ST13",
          "title": "The Empathetic Listen",
          "desc":
              "Listen to someone venting and provide a supportive response.",
        },
        {
          "id": "ST14",
          "title": "The Public Presentation",
          "desc":
              "Speak for 1-2 minutes about a topic you love in a social gathering.",
        },
        {
          "id": "ST15",
          "title": "The Vulnerable Share",
          "desc":
              "Admit to a group that you were nervous about something, and laugh about it.",
        },
        {
          "id": "ST16",
          "title": "The Boundary Set",
          "desc":
              "Politely decline an invitation you don't want to attend without over-explaining.",
        },
        {
          "id": "ST17",
          "title": "The Active Mediator",
          "desc": "Help two people find a middle ground in a disagreement.",
        },
        {
          "id": "ST18",
          "title": "The Public Compliment",
          "desc": "Publicly praise someone's effort or achievement in a group.",
        },
        {
          "id": "ST19",
          "title": "The Direct Approach",
          "desc":
              "Ask someone directly for a favor or a piece of advice you need.",
        },
        {
          "id": "ST20",
          "title": "The Conversation Pivot",
          "desc":
              "Smoothly transition a conversation from a boring topic to an interesting one.",
        },
      ],
      "Bloom": [
        {
          "id": "B1",
          "title": "The Gift",
          "desc":
              "Give a small treat to someone and say 'I thought you'd like this'.",
        },
        {
          "id": "B2",
          "title": "The Bold Lead",
          "desc":
              "Suggest a plan or a place to visit to a small group of people.",
        },
        {
          "id": "B3",
          "title": "The Appreciation",
          "desc":
              "Tell someone specifically why you appreciate having them in your life.",
        },
        {
          "id": "B4",
          "title": "The Social Host",
          "desc":
              "Organize a small get-together or a coffee date for a few people.",
        },
        {
          "id": "B5",
          "title": "The Deep Dive",
          "desc":
              "Have a deep, meaningful conversation with someone for over 15 minutes.",
        },
        {
          "id": "B6",
          "title": "The Confidence Peak",
          "desc": "Initiate a conversation with someone you find intimidating.",
        },
        {
          "id": "B7",
          "title": "The Public Toast",
          "desc":
              "Make a short, positive toast or shout-out to someone in a group.",
        },
        {
          "id": "B8",
          "title": "The Boundary Setter",
          "desc":
              "Say 'No' to a request firmly but kindly, without over-explaining.",
        },
        {
          "id": "B9",
          "title": "The Direct Request",
          "desc": "Ask someone you admire for a 10-minute chat or mentorship.",
        },
        {
          "id": "B10",
          "title": "The Emotional Lead",
          "desc":
              "Initiate a conversation about feelings or mental health with a friend.",
        },
        {
          "id": "B11",
          "title": "The Social Mediator",
          "desc":
              "Help two people resolve a small conflict through a calm conversation.",
        },
        {
          "id": "B12",
          "title": "The Bold Compliment",
          "desc":
              "Tell a complete stranger something you genuinely admire about them.",
        },
        {
          "id": "B13",
          "title": "The Networking Move",
          "desc":
              "Introduce yourself to a professional in your field and ask for advice.",
        },
        {
          "id": "B14",
          "title": "The Courageous Truth",
          "desc":
              "Tell someone a truth that is difficult but helpful for the relationship.",
        },
        {
          "id": "B15",
          "title": "The Full Bloom",
          "desc":
              "Host a small social event and make sure every guest feels welcome.",
        },
        {
          "id": "B16",
          "title": "The Public Speaker",
          "desc":
              "Volunteer to speak or lead a small part of a meeting or event.",
        },
        {
          "id": "B17",
          "title": "The Vulnerable Lead",
          "desc": "Share a struggle you've overcome to encourage someone else.",
        },
        {
          "id": "B18",
          "title": "The Bold Apology",
          "desc":
              "Initiate a conversation to apologize for a past mistake, even if it was long ago.",
        },
        {
          "id": "B19",
          "title": "The Mentor",
          "desc":
              "Offer to help someone who is less experienced than you with a skill.",
        },
        {
          "id": "B20",
          "title": "The Social Architect",
          "desc":
              "Create a new social tradition or a recurring meetup for a group of friends.",
        },
      ],
    },
    'es': {
      "Seedling": [
        {
          "id": "S1",
          "title": "El Primer Paso",
          "desc": "Mantén contacto visual y sonríe a una persona hoy.",
        },
        {
          "id": "S2",
          "title": "Un Hola Simple",
          "desc": "Di 'Buenos días' o 'Hola' a un vecino.",
        },
        {
          "id": "S3",
          "title": "El Gracias",
          "desc": "Di 'Gracias' claramente a un tendero.",
        },
        {
          "id": "S4",
          "title": "La Observación",
          "desc": "Nota algo positivo sobre un extraño y sonríe.",
        },
        {
          "id": "S5",
          "title": "El Saludo Silencioso",
          "desc":
              "Saluuda con la mano a alguien que reconozcas a la distancia.",
        },
        {
          "id": "S6",
          "title": "Sostener la Puerta",
          "desc": "Sostén la puerta abierta para alguien detrás de ti.",
        },
        {
          "id": "S7",
          "title": "El Asentimiento",
          "desc": "Haz un gesto amable con la cabeza a un colega al pasar.",
        },
        {
          "id": "S8",
          "title": "El Espejo",
          "desc":
              "Practica tu 'sonrisa confiada' frente al espejo durante 1 minuto.",
        },
        {
          "id": "S9",
          "title": "La Mirada Breve",
          "desc":
              "Mira a alguien durante 2 segundos, luego sonríe y mira hacia otro lado.",
        },
        {
          "id": "S10",
          "title": "El Elogio Silencioso",
          "desc":
              "Escribe un comentario amable en la publicación de alguien en redes sociales.",
        },
        {
          "id": "S11",
          "title": "Compartir el Espacio",
          "desc":
              "Siéntate junto a alguien en un área pública sin apartar la mirada inmediatamente.",
        },
        {
          "id": "S12",
          "title": "El Reconocimiento Simple",
          "desc":
              "Di 'Disculpe' educadamente al pasar junto a alguien en un pasillo.",
        },
        {
          "id": "S13",
          "title": "El Saludo Cálido",
          "desc": "Di 'Hola' a un repartidor or mensajero.",
        },
        {
          "id": "S14",
          "title": "El Pequeño Saludo",
          "desc":
              "Saluuda con la mano a un niño or a una mascota (con permiso del dueño).",
        },
        {
          "id": "S15",
          "title": "La Sonrisa Suave",
          "desc": "Sonríe a tres personas diferentes hoy.",
        },
        {
          "id": "S16",
          "title": "El Reto del Contacto Visual",
          "desc":
              "Mantén el contacto visual con un cajero hasta que él mire hacia otro lado primero.",
        },
        {
          "id": "S17",
          "title": "La Respiración Suave",
          "desc":
              "Toma 3 respiraciones profundas antes de entrar en un espacio social hoy.",
        },
        {
          "id": "S18",
          "title": "La Presencia",
          "desc":
              "Quédate en un área concurrida durante 5 minutos sin mirar tu teléfono.",
        },
        {
          "id": "S19",
          "title": "El Asentimiento Casual",
          "desc":
              "Asiente con la cabeza a un extraño que haga contacto visual contigo.",
        },
        {
          "id": "S20",
          "title": "La Voz Suave",
          "desc":
              "Dile 'Que tenga un buen día' a alguien al salir de una tienda.",
        },
      ],
      "Sprout": [
        {
          "id": "SP1",
          "title": "El Cumplido",
          "desc": "Dale un cumplido genuino a un colega or compañero de clase.",
        },
        {
          "id": "SP2",
          "title": "La Pregunta",
          "desc": "Pregunta a un extraño la hora or direcciones.",
        },
        {
          "id": "SP3",
          "title": "Charla Casual",
          "desc":
              "Pregunta a alguien '¿Cómo va tu día?' y escucha la respuesta.",
        },
        {
          "id": "SP4",
          "title": "La Petición",
          "desc":
              "Pide ayuda a un empleado de la tienda para encontrar un artículo específico.",
        },
        {
          "id": "SP5",
          "title": "El Pedido",
          "desc":
              "Pide una bebida or comida y pregunta al personal cómo están.",
        },
        {
          "id": "SP6",
          "title": "El Saludo",
          "desc": "Preséntate a alguien nuevo en tu área.",
        },
        {
          "id": "SP7",
          "title": "Charla sobre el Clima",
          "desc": "Menciona el clima a alguien mientras esperas en una fila.",
        },
        {
          "id": "SP8",
          "title": "La Consulta Simple",
          "desc": "Pregunta a un compañero '¿Qué hiciste el fin de semana?'",
        },
        {
          "id": "SP9",
          "title": "El Ofrecimiento de Ayuda",
          "desc":
              "Pregunta a alguien '¿Necesitas ayuda con eso?' si parece estar luchando.",
        },
        {
          "id": "SP10",
          "title": "La Opinión",
          "desc":
              "Pregunta a un amigo '¿Qué piensas de esto?' sobre un objeto pequeño.",
        },
        {
          "id": "SP11",
          "title": "La Confirmación",
          "desc":
              "Confirma un detalle con un extraño (ej., '¿Es esta la fila correcta?').",
        },
        {
          "id": "SP12",
          "title": "El Espacio Compartido",
          "desc":
              "Haz un comentario pequeño sobre el entorno (ej., 'Está muy lleno hoy').",
        },
        {
          "id": "SP13",
          "title": "El Pequeño Favor",
          "desc":
              "Pide a alguien que te pase algo (como una servilleta) en una mesa.",
        },
        {
          "id": "SP14",
          "title": "La Retroalimentación Cálida",
          "desc": "Dile a un mesero que la comida estuvo genial antes de irte.",
        },
        {
          "id": "SP15",
          "title": "El Saludo Casual",
          "desc":
              "Envía un texto de '¿Cómo estás?' a alguien con quien no has hablado en un mes.",
        },
        {
          "id": "SP16",
          "title": "La Pregunta Abierta",
          "desc":
              "Pregunta a alguien '¿Cuál es tu lugar favorito para visitar en esta ciudad?'",
        },
        {
          "id": "SP17",
          "title": "El Riesgo Más Pequeño",
          "desc":
              "Pregunta a un extraño si sabe dónde está el baño más cercano.",
        },
        {
          "id": "SP18",
          "title": "El Elogio al Objeto",
          "desc": "Dile a alguien que te gustan sus zapatos/bolso/accesorio.",
        },
        {
          "id": "SP19",
          "title": "La Pausa Educada",
          "desc":
              "Espera a que alguien termine de hablar completamente antes de responderle.",
        },
        {
          "id": "SP20",
          "title": "El Saludo Amistoso",
          "desc":
              "Saluuda con la mano y di 'Adiós' a alguien con quien acabas de tener una breve interacción.",
        },
      ],
      "Leaf": [
        {
          "id": "L1",
          "title": "Buscador de Opiniones",
          "desc":
              "Pregunta a alguien su opinión sobre un libro, película or canción.",
        },
        {
          "id": "L2",
          "title": "El Detalle",
          "desc":
              "Haz una pregunta de seguimiento después de que alguien te cuente algo sobre sí mismo.",
        },
        {
          "id": "L3",
          "title": "La Recomendación",
          "desc":
              "Pregunta a un extraño por una recomendación de un buen lugar para comer cerca.",
        },
        {
          "id": "L4",
          "title": "La Conexión",
          "desc":
              "Encuentra un interés común con alguien y habla de ello durante 2 minutos.",
        },
        {
          "id": "L5",
          "title": "La Mano Ayudadora",
          "desc":
              "Ofrece ayuda a alguien con una tarea pequeña (como cargar una bolsa).",
        },
        {
          "id": "L6",
          "title": "La Observación Social",
          "desc":
              "Inicia una conversación basada en algo que esté sucediendo alrededor de ambos.",
        },
        {
          "id": "L7",
          "title": "La Pregunta Abierta",
          "desc": "Pregunta a alguien '¿Cómo llegaste a este campo laboral?'",
        },
        {
          "id": "L8",
          "title": "El Oyente Activo",
          "desc":
              "Escucha a alguien durante 3 minutos sin interrumpir, luego resume lo que dijo.",
        },
        {
          "id": "L9",
          "title": "La Risa Compartida",
          "desc":
              "Cuenta una historia corta y divertida or un chiste a un grupo pequeño.",
        },
        {
          "id": "L10",
          "title": "La Curiosidad",
          "desc": "Pregunta a alguien de dónde es y qué le gusta de ese lugar.",
        },
        {
          "id": "L11",
          "title": "El Interés Sincero",
          "desc":
              "Pregunta a un colega sobre sus pasatiempos fuera del trabajo.",
        },
        {
          "id": "L12",
          "title": "El Consejo Suave",
          "desc":
              "Dale a alguien un consejo útil sobre algo en lo que seas bueno.",
        },
        {
          "id": "L13",
          "title": "El Asentimiento Grupal",
          "desc":
              "Estar de acuerdo con el punto de alguien en una discusión de grupo pequeño.",
        },
        {
          "id": "L14",
          "title": "La Invitación Casual",
          "desc": "Pregunta a alguien '¿Te gustaría acompañarnos a almorzar?'",
        },
        {
          "id": "L15",
          "title": "La Reflexión Honesta",
          "desc":
              "Dile a alguien 'Realmente aprecié cuando hiciste X' y explica por qué.",
        },
        {
          "id": "L16",
          "title": "La Brecha de Curiosidad",
          "desc":
              "Pregunta a alguien 'Siempre me he preguntado, ¿cómo funciona X realmente?'",
        },
        {
          "id": "L17",
          "title": "Liderazgo de Grupo Pequeño",
          "desc":
              "Haz una pregunta que requiera que 2 or 3 personas de un grupo respondan.",
        },
        {
          "id": "L18",
          "title": "El Cumplido Genuino",
          "desc":
              "Elogia a alguien por un rasgo de su personalidad (ej., 'Eres un gran orador').",
        },
        {
          "id": "L19",
          "title": "La Experiencia Compartida",
          "desc":
              "Di 'Yo he estado en esa situación también' durante una conversación.",
        },
        {
          "id": "L20",
          "title": "La Pausa Significativa",
          "desc":
              "Permite que ocurra un silencio en una conversación sin apresurarse a llenarlo.",
        },
      ],
      "Stem": [
        {
          "id": "ST1",
          "title": "El Inicio Valiente",
          "desc": "Inicia una conversación con alguien que no conozcas bien.",
        },
        {
          "id": "ST2",
          "title": "El Compartir Honesto",
          "desc":
              "Comparte una pequeña historia personal u opinión en un entorno grupal.",
        },
        {
          "id": "ST3",
          "title": "El Debate",
          "desc":
              "No estés de acuerdo educadamente con la opinión de alguien y explica por qué.",
        },
        {
          "id": "ST4",
          "title": "La Entrada al Grupo",
          "desc":
              "Únete a una conversación grupal y aporta una frase reflexiva.",
        },
        {
          "id": "ST5",
          "title": "El Líder del Tema",
          "desc": "Plantea un nuevo tema de conversación en un grupo social.",
        },
        {
          "id": "ST6",
          "title": "La Pregunta Pública",
          "desc":
              "Haz una pregunta en una reunión pública or en un salón de clases.",
        },
        {
          "id": "ST7",
          "title": "La Petición Audaz",
          "desc":
              "Pregunta a un extraño si puedes sentarte junto a él en un café or parque.",
        },
        {
          "id": "ST8",
          "title": "El Puente de Conversación",
          "desc":
              "Presenta a dos personas que no se conocen y encuentra algo en común.",
        },
        {
          "id": "ST9",
          "title": "La Necesidad Asertiva",
          "desc":
              "Pide educadamente a alguien que se mueva or deje de hacer algo que te moleste.",
        },
        {
          "id": "ST10",
          "title": "El Cuentacuentos",
          "desc":
              "Toma la iniciativa de contar una historia a un grupo de 3 or más personas.",
        },
        {
          "id": "ST11",
          "title": "El Desafío Abierto",
          "desc":
              "Cuestiona una opinión común en un grupo de manera amistosa y respetuosa.",
        },
        {
          "id": "ST12",
          "title": "La Iniciativa Social",
          "desc":
              "Sé la primera persona en decir 'Hola' a todos al entrar en una habitación.",
        },
        {
          "id": "ST13",
          "title": "La Escucha Empática",
          "desc":
              "Escucha a alguien desahogarse y proporciona una respuesta comprensiva.",
        },
        {
          "id": "ST14",
          "title": "La Presentación Pública",
          "desc":
              "Habla durante 1-2 minutos sobre un tema que ames en una reunión social.",
        },
        {
          "id": "ST15",
          "title": "El Compartir Vulnerable",
          "desc":
              "Admite ante un grupo que estabas nervioso por algo y ríete de ello.",
        },
        {
          "id": "ST16",
          "title": "El Límite Establecido",
          "desc":
              "Rechaza educadamente una invitación que no quieres aceptar sin dar demasiadas explicaciones.",
        },
        {
          "id": "ST17",
          "title": "El Mediador Activo",
          "desc":
              "Ayuda a dos personas a encontrar un punto medio en un desacuerdo.",
        },
        {
          "id": "ST18",
          "title": "El Cumplido Público",
          "desc":
              "Elogia públicamente el esfuerzo or logro de alguien en un grupo.",
        },
        {
          "id": "ST19",
          "title": "El Enfoque Directo",
          "desc":
              "Pide a alguien directamente un favor or un consejo que necesites.",
        },
        {
          "id": "ST20",
          "title": "El Pivote de Conversación",
          "desc":
              "Transiciona suavemente una conversación de un tema aburrido a uno interesante.",
        },
      ],
      "Bloom": [
        {
          "id": "B1",
          "title": "El Regalo",
          "desc":
              "Dale un dulce a alguien y dile 'Pensé que te gustaría esto'.",
        },
        {
          "id": "B2",
          "title": "El Liderazgo Audaz",
          "desc":
              "Sugiere un plan or un lugar para visitar a un grupo pequeño de personas.",
        },
        {
          "id": "B3",
          "title": "El Aprecio",
          "desc":
              "Dile a alguien específicamente por qué aprecias tenerlo en tu vida.",
        },
        {
          "id": "B4",
          "title": "El Anfitrión Social",
          "desc":
              "Organiza una pequeña reunión or una cita para tomar café con algunas personas.",
        },
        {
          "id": "B5",
          "title": "La Inmersión Profunda",
          "desc":
              "Tén una conversación profunda y significativa con alguien durante más de 15 minutos.",
        },
        {
          "id": "B6",
          "title": "El Pico de Confianza",
          "desc":
              "Inicia una conversación con alguien que consideres intimidante.",
        },
        {
          "id": "B7",
          "title": "El Brindis Público",
          "desc":
              "Haz un brindis corto y positivo or un reconocimiento a alguien en un grupo.",
        },
        {
          "id": "B8",
          "title": "El Establecedor de Límites",
          "desc":
              "Di 'No' a una petición con firmeza pero amablemente, sin dar demasiadas explicaciones.",
        },
        {
          "id": "B9",
          "title": "La Petición Directa",
          "desc":
              "Pide a alguien que admires una charla de 10 minutos or mentoría.",
        },
        {
          "id": "B10",
          "title": "El Liderazgo Emocional",
          "desc":
              "Inicia una conversación sobre sentimientos or salud mental con un amigo.",
        },
        {
          "id": "B11",
          "title": "El Mediador Social",
          "desc":
              "Ayuda a dos personas a resolver un conflicto pequeño a través de una conversación calmada.",
        },
        {
          "id": "B12",
          "title": "El Cumplido Audaz",
          "desc":
              "Dile a un completo extraño algo que genuinamente admires de ellos.",
        },
        {
          "id": "B13",
          "title": "El Movimiento de Networking",
          "desc": "Preséntate a un profesional de tu campo y pide consejo.",
        },
        {
          "id": "B14",
          "title": "La Verdad Valiente",
          "desc":
              "Dile a alguien una verdad que sea difícil de decir, pero útil para la relación.",
        },
        {
          "id": "B15",
          "title": "El Florecimiento Total",
          "desc":
              "Organiza un evento social pequeño y asegúrate de que cada invitado se sienta bienvenido.",
        },
        {
          "id": "B16",
          "title": "El Orador Público",
          "desc":
              "Ofrécete para hablar or dirigir una pequeña parte de una reunión or evento.",
        },
        {
          "id": "B17",
          "title": "El Liderazgo Vulnerable",
          "desc":
              "Comparte una lucha que hayas superCidado para animar a alguien más.",
        },
        {
          "id": "B18",
          "title": "La Disculpa Audaz",
          "desc":
              "Inicia una conversación para disculparte por un error pasado, incluso si fue hace mucho tiempo.",
        },
        {
          "id": "B19",
          "title": "El Mentor",
          "desc":
              "Ofrece ayuda a alguien que tenga menos experiencia que tú en una habilidad.",
        },
        {
          "id": "B20",
          "title": "El Arquitecto Social",
          "desc":
              "Crea una nueva tradición social or una reunión recurrente para un grupo de amigos.",
        },
      ],
    },
    'fr': {
      "Seedling": [
        {
          "id": "S1",
          "title": "Le Premier Pas",
          "desc":
              "Établissez un contact visuel et souriez à une personne aujourd'hui.",
        },
        {
          "id": "S2",
          "title": "Un Bonjour Simple",
          "desc": "Dites 'Bonjour' à un voisin.",
        },
        {
          "id": "S3",
          "title": "Le Merci",
          "desc": "Dites 'Merci' clairement à un commerçant.",
        },
        {
          "id": "S4",
          "title": "L'Observation",
          "desc":
              "Remarquez quelque chose de positif chez un étranger et souriez.",
        },
        {
          "id": "S5",
          "title": "Le Salut Silencieux",
          "desc":
              "Faites un signe de la main à quelqu'un que vous reconnaissez au loin.",
        },
        {
          "id": "S6",
          "title": "Tenir la Porte",
          "desc": "Tenez la porte ouverte pour quelqu'un derrière vous.",
        },
        {
          "id": "S7",
          "title": "Le Hochement de Tête",
          "desc": "Faites un signe de tête amical à un collègue en passant.",
        },
        {
          "id": "S8",
          "title": "Le Miroir",
          "desc":
              "Pratiquez votre 'sourire confiant' dans le miroir pendant 1 minute.",
        },
        {
          "id": "S9",
          "title": "Le Regard Bref",
          "desc":
              "Regardez quelqu'un pendant 2 secondes, puis souriez et détournez le regard.",
        },
        {
          "id": "S10",
          "title": "L'Éloge Silencieux",
          "desc":
              "Écrivez un commentaire gentil sur la publication de quelqu'un sur les réseaux sociaux.",
        },
        {
          "id": "S11",
          "title": "Partager l'Espace",
          "desc":
              "Asseyez-vous à côté de quelqu'un dans un lieu public sans détourner le regard immédiatement.",
        },
        {
          "id": "S12",
          "title": "L'Héritage Simple",
          "desc":
              "Dites 'Excusez-moi' poliment en passant devant quelqu'un dans un couloir.",
        },
        {
          "id": "S13",
          "title": "Le Salut Chaleureux",
          "desc": "Dites 'Salut' à un livreur or un coursier.",
        },
        {
          "id": "S14",
          "title": "Le Petit Salut",
          "desc":
              "Faites un signe de la main à un enfant or à un animal (avec la permission du propriétaire).",
        },
        {
          "id": "S15",
          "title": "Le Sourire Doux",
          "desc": "Sourire à trois personnes différentes aujourd'hui.",
        },
        {
          "id": "S16",
          "title": "Le Défi du Contact Visuel",
          "desc":
              "Maintenez le contact visuel avec un caissier jusqu'à ce qu'il détourne le regard en premier.",
        },
        {
          "id": "S17",
          "title": "La Respiration Douce",
          "desc":
              "Prenez 3 respirations profondes avant d'entrer dans un espace social aujourd'hui.",
        },
        {
          "id": "S18",
          "title": "La Présence",
          "desc":
              "Tenez-vous dans un endroit bondé pendant 5 minutes sans regarder votre téléphone.",
        },
        {
          "id": "S19",
          "title": "Le Hochement Casual",
          "desc":
              "Faites un signe de tête à un étranger qui établit un contact visuel avec vous.",
        },
        {
          "id": "S20",
          "title": "La Voix Douce",
          "desc": "Dites 'Bonne journée' à quelqu'un en quittant un magasin.",
        },
      ],
      "Sprout": [
        {
          "id": "SP1",
          "title": "Le Compliment",
          "desc":
              "Faites un compliment sincère à un collègue or un camarade de classe.",
        },
        {
          "id": "SP2",
          "title": "La Question",
          "desc": "Demandez l'heure or des directions à un étranger.",
        },
        {
          "id": "SP3",
          "title": "Conversation Banale",
          "desc":
              "Demandez à quelqu'un 'Comment se passe votre journée ?' et écoutez la réponse.",
        },
        {
          "id": "SP4",
          "title": "La Demande",
          "desc":
              "Demandez l'aide d'un employé de magasin pour trouver un article spécifique.",
        },
        {
          "id": "SP5",
          "title": "La Commande",
          "desc":
              "Commandez une boisson or de la nourriture et demandez au personnel comment ils vont.",
        },
        {
          "id": "SP6",
          "title": "La Salutation",
          "desc": "Présentez-vous à quelqu'un de nouveau dans votre quartier.",
        },
        {
          "id": "SP7",
          "title": "Parler du Temps",
          "desc": "Mentionnez le temps à quelqu'un en attendant dans une file.",
        },
        {
          "id": "SP8",
          "title": "L'Interrogation Simple",
          "desc": "Demandez à un collègue 'Qu'as-tu fait ce week-end ?'",
        },
        {
          "id": "SP9",
          "title": "L'Offre d'Aide",
          "desc":
              "Demandez à quelqu'un 'Avez-vous besoin d'aide ?' s'il semble être en difficulté.",
        },
        {
          "id": "SP10",
          "title": "L'Opinion",
          "desc": "Demandez or l'avis d'un ami sur un petit objet.",
        },
        {
          "id": "SP11",
          "title": "La Confirmation",
          "desc":
              "Confirmez un détail avec un étranger (ex: 'Est-ce la bonne file ?').",
        },
        {
          "id": "SP12",
          "title": "L'Espace Partagé",
          "desc":
              "Faites un petit commentaire sur l'environnement (ex: 'C'est vraiment bondé aujourd'hui').",
        },
        {
          "id": "SP13",
          "title": "Le Petit Service",
          "desc":
              "Demandez à quelqu'un de vous passer quelque chose (comme une serviette) à une table.",
        },
        {
          "id": "SP14",
          "title": "Le Retour Chaleureux",
          "desc":
              "Dites à un serveur que la nourriture était excellente avant de partir.",
        },
        {
          "id": "SP15",
          "title": "Le Petit Message",
          "desc":
              "Envoyez un texte 'Comment vas-tu ?' à quelqu'un avec qui vous n'avez pas parlé depuis un mois.",
        },
        {
          "id": "SP16",
          "title": "La Question Ouverte",
          "desc":
              "Demandez à quelqu'un 'Quel est votre endroit préféré à visiter dans cette ville ?'",
        },
        {
          "id": "SP17",
          "title": "Le Plus Petit Risque",
          "desc":
              "Demandez à un étranger s'il sait où se trouvent les toilettes les plus proches.",
        },
        {
          "id": "SP18",
          "title": "L'Éloge de l'Objet",
          "desc":
              "Dites à quelqu'un que vous aimez ses chaussures/son sac/son accessoire.",
        },
        {
          "id": "SP19",
          "title": "La Pause Polie",
          "desc":
              "Attendez que quelqu'un ait fini de parler complètement avant de lui répondre.",
        },
        {
          "id": "SP20",
          "title": "Le Salut Amical",
          "desc":
              "Faites un signe de la main et dites 'Au revoir' à quelqu'un avec qui vous venez d'avoir une brève interaction.",
        },
      ],
      "Leaf": [
        {
          "id": "L1",
          "title": "Chercheur d'Opinions",
          "desc":
              "Demandez l'avis de quelqu'un sur un livre, un film or une chanson.",
        },
        {
          "id": "L2",
          "title": "Le Détail",
          "desc":
              "Posez une question de suivi après que quelqu'un vous ait parlé de lui-même.",
        },
        {
          "id": "L3",
          "title": "La Recommandation",
          "desc":
              "Demandez à un étranger une recommandation pour un bon endroit où manger à proximité.",
        },
        {
          "id": "L4",
          "title": "La Connexion",
          "desc":
              "Trouvez un centre d'intérêt commun avec quelqu'un et parlez-en pendant 2 minutes.",
        },
        {
          "id": "L5",
          "title": "La Main Secourable",
          "desc":
              "Proposez d'aider quelqu'un pour une petite tâche (comme porter un sac).",
        },
        {
          "id": "L6",
          "title": "L'Observation Sociale",
          "desc":
              "Engagez la conversation en vous basant sur quelque chose qui se passe autour de vous.",
        },
        {
          "id": "L7",
          "title": "La Question Ouverte",
          "desc":
              "Demandez à quelqu'un 'Comment vous êtes-vous lancé dans ce métier ?'",
        },
        {
          "id": "L8",
          "title": "L'Écoute Active",
          "desc":
              "Écoutez quelqu'un pendant 3 minutes sans l'interrompre, puis résumez ce qu'il a dit.",
        },
        {
          "id": "L9",
          "title": "Le Rire Partagé",
          "desc":
              "Racontez une courte histoire drôle or une blague à un petit groupe.",
        },
        {
          "id": "L10",
          "title": "La Curiosité",
          "desc":
              "Demandez à quelqu'un d'où il vient et ce qu'il aime dans cet endroit.",
        },
        {
          "id": "L11",
          "title": "L'Intérêt Sincère",
          "desc":
              "Interrogez un collègue sur ses loisirs en dehors du travail.",
        },
        {
          "id": "L12",
          "title": "Le Conseil Doux",
          "desc":
              "Donnez un conseil utile à quelqu'un sur un sujet où vous êtes doué.",
        },
        {
          "id": "L13",
          "title": "L'Accord Groupal",
          "desc":
              "Approuvez le point de vue de quelqu'un lors d'une discussion de groupe.",
        },
        {
          "id": "L14",
          "title": "L'Invitation Casual",
          "desc":
              "Demandez à quelqu'un 'Voudriez-vous nous accompagner pour déjeuner ?'",
        },
        {
          "id": "L15",
          "title": "L'Honnête Réflexion",
          "desc":
              "Dites à quelqu'un 'J'ai vraiment apprécié quand tu as fait X' et expliquez pourquoi.",
        },
        {
          "id": "L16",
          "title": "La Brèche de Curiosité",
          "desc":
              "Demandez à quelqu'un 'Je me suis toujours demandé, comment X fonctionne-t-il vraiment ?'",
        },
        {
          "id": "L17",
          "title": "Liderazgo de Grupo Pequeño",
          "desc":
              "Posez une question qui nécessite que 2 ou 3 personnes d'un groupe répondent.",
        },
        {
          "id": "L18",
          "title": "Le Compliment Genuin",
          "desc":
              "Faites un compliment à quelqu'un sur un trait de sa personnalité (ex: 'Vous êtes un excellent auditeur').",
        },
        {
          "id": "L19",
          "title": "L'Expérience Partagée",
          "desc":
              "Dites 'J'ai déjà été dans cette situation aussi' lors d'une conversation.",
        },
        {
          "id": "L20",
          "title": "La Pause Significative",
          "desc":
              "Laissez un silence s'installer dans une conversation sans se précipiter pour le combler.",
        },
      ],
      "Stem": [
        {
          "id": "ST1",
          "title": "Le Début Courageux",
          "desc":
              "Engagez la conversation avec quelqu'un que vous connaissez peu.",
        },
        {
          "id": "ST2",
          "title": "Le Partage Honnête",
          "desc":
              "Partagez une petite histoire personnelle ou une opinion dans un cadre groupal.",
        },
        {
          "id": "ST3",
          "title": "Le Débat",
          "desc":
              "Exprimez poliment votre désaccord avec l'opinion de quelqu'un et expliquez pourquoi.",
        },
        {
          "id": "ST4",
          "title": "L'Entrée dans le Groupe",
          "desc":
              "Joignez-vous à une conversation de groupe et apportez une phrase réfléchie.",
        },
        {
          "id": "ST5",
          "title": "Le Leader du Sujet",
          "desc":
              "Lancez un nouveau sujet de conversation dans un groupe social.",
        },
        {
          "id": "ST6",
          "title": "La Question Publique",
          "desc":
              "Posez une question lors d'une réunion publique ou dans une salle de classe.",
        },
        {
          "id": "ST7",
          "title": "La Demande Audacieuse",
          "desc":
              "Demandez à un étranger si vous pouvez vous asseoir à côté de lui dans un café ou un parc.",
        },
        {
          "id": "ST8",
          "title": "Le Pont de Conversation",
          "desc":
              "Présentez deux personnes qui ne se connaissent pas et trouvez un point commun.",
        },
        {
          "id": "ST9",
          "title": "Le Besoin Assertif",
          "desc":
              "Demandez poliment à quelqu'un de se déplacer ou d'arrêter de faire quelque chose qui vous gêne.",
        },
        {
          "id": "ST10",
          "title": "Le Conteur",
          "desc":
              "Prenez l'initiative de raconter une histoire à un groupe de 3 personnes ou plus.",
        },
        {
          "id": "ST11",
          "title": "Le Défi Ouvert",
          "desc":
              "Remettez en question une opinion commune dans un groupe de manière amicale et respectueuse.",
        },
        {
          "id": "ST12",
          "title": "L'Initiative Sociale",
          "desc":
              "Soyez la première personne à dire 'Bonjour' à tout le monde en entrant dans une pièce.",
        },
        {
          "id": "ST13",
          "title": "L'Écoute Empathique",
          "desc":
              "Écoutez quelqu'un se confier et apportez une réponse compréhensive.",
        },
        {
          "id": "ST14",
          "title": "La Présentation Publique",
          "desc":
              "Parlez pendant 1 à 2 minutes d'un sujet que vous aimez lors d'un rassemblement social.",
        },
        {
          "id": "ST15",
          "title": "Le Partage Vulnérable",
          "desc":
              "Admettez devant un groupe que vous étiez nerveux pour quelque chose, et riez-en.",
        },
        {
          "id": "ST16",
          "title": "La Limite Établie",
          "desc":
              "Refusez poliment une invitation que vous ne souhaitez pas accepter sans trop vous justifier.",
        },
        {
          "id": "ST17",
          "title": "Le Médiateur Actif",
          "desc":
              "Aidez deux personnes à trouver un terrain d'entente lors d'un désaccord.",
        },
        {
          "id": "ST18",
          "title": "Le Compliment Public",
          "desc":
              "Louvez publiquement l'effort ou la réussite de quelqu'un dans un groupe.",
        },
        {
          "id": "ST19",
          "title": "L'Approche Directe",
          "desc":
              "Demandez directement à quelqu'un une faveur or un conseil dont vous avez besoin.",
        },
        {
          "id": "ST20",
          "title": "Le Pivot de Conversation",
          "desc":
              "Faites glisser doucement une conversation d'un sujet ennuyeux vers un sujet intéressant.",
        },
      ],
      "Bloom": [
        {
          "id": "B1",
          "title": "Le Cadeau",
          "desc":
              "Offrez une petite friandise à quelqu'un en disant 'J'ai pensé que cela vous plairait'.",
        },
        {
          "id": "B2",
          "title": "Le Lead Audacieux",
          "desc":
              "Suggérez un plan or un lieu à visiter à un petit groupe de personnes.",
        },
        {
          "id": "B3",
          "title": "Le Appréciation",
          "desc":
              "Dites à quelqu'un spécifiquement pourquoi vous appréciez sa présence dans votre vie.",
        },
        {
          "id": "B4",
          "title": "Le Hôte Social",
          "desc":
              "Organisez un petit rassemblement or un rendez-vous café pour quelques personnes.",
        },
        {
          "id": "B5",
          "title": "Le Immersion Profonde",
          "desc":
              "Ayez une conversation profonde et significative avec quelqu'un pendant plus de 15 minutes.",
        },
        {
          "id": "B6",
          "title": "Le Pic de Confiance",
          "desc":
              "Engagez la conversation avec quelqu'un que vous trouvez intimidant.",
        },
        {
          "id": "B7",
          "title": "Le Toast Public",
          "desc":
              "Faites un toast court et positif or un hommage à quelqu'un dans un groupe.",
        },
        {
          "id": "B8",
          "title": "Le Établisseur de Limites",
          "desc":
              "Dites 'Non' à une demande avec fermeté mais gentillesse, sans trop vous justifier.",
        },
        {
          "id": "B9",
          "title": "La Demande Directe",
          "desc":
              "Demandez à quelqu'un que vous admirez un entretien de 10 minutes or un mentorat.",
        },
        {
          "id": "B10",
          "title": "Le Lead Émotionnel",
          "desc":
              "Engagez une conversation sur les sentiments or la santé mentale avec un ami.",
        },
        {
          "id": "B11",
          "title": "Le Médiateur Social",
          "desc":
              "Aidez deux personnes à résoudre un petit conflit via une conversation calme.",
        },
        {
          "id": "B12",
          "title": "Le Compliment Audacieux",
          "desc":
              "Dites à un parfait étranger quelque chose que vous admirez sincèrement chez lui.",
        },
        {
          "id": "B13",
          "title": "Le Mouvement de Networking",
          "desc":
              "Présentez-vous à un professionnel de votre domaine et demandez-lui conseil.",
        },
        {
          "id": "B14",
          "title": "La Vérité Courageuse",
          "desc":
              "Dites à quelqu'un une vérité difficile à dire, mais utile pour la relation.",
        },
        {
          "id": "B15",
          "title": "Le Épanouissement Total",
          "desc":
              "Organisez un petit événement social et assurez-vous que chaque invité se sente accueilli.",
        },
        {
          "id": "B16",
          "title": "Le Orateur Public",
          "desc":
              "Proposez-vous pour parler or diriger une petite partie d'une réunion or d'un événement.",
        },
        {
          "id": "B17",
          "title": "Le Lead Vulnérable",
          "desc":
              "Partagez une lutte que vous avez surmontée pour encourager quelqu'un d'autre.",
        },
        {
          "id": "B18",
          "title": "Le Excuse Audacieuse",
          "desc":
              "Engagez une conversation pour vous excuser d'une erreur passée, même si c'était il y a longtemps.",
        },
        {
          "id": "B19",
          "title": "Le Mentor",
          "desc":
              "Offrez votre aide à quelqu'un qui a moins d'expérience que vous dans une compétence.",
        },
        {
          "id": "B20",
          "title": "Le Architecte Social",
          "desc":
              "Créez une nouvelle tradition sociale or une réunion récurrente pour un groupe d'amis.",
        },
      ],
    },
    'hi': {
      "Seedling": [
        {
          "id": "S1",
          "title": "पहला कदम",
          "desc": "आज किसी एक व्यक्ति से नज़रें मिलाएं और मुस्कुराएं।",
        },
        {
          "id": "S2",
          "title": "एक साधारण नमस्ते",
          "desc": "किसी पड़ोसी को 'सुप्रभात' या 'नमस्ते' कहें।",
        },
        {
          "id": "S3",
          "title": "धन्यवाद",
          "desc": "दुकानदार को स्पष्ट रूप से 'धन्यवाद' कहें।",
        },
        {
          "id": "S4",
          "title": "अवलोकन",
          "desc": "किसी अजनबी में कुछ सकारात्मक देखें और मुस्कुराएं।",
        },
        {
          "id": "S5",
          "title": "शांत इशारा",
          "desc": "दूरी से किसी ऐसे व्यक्ति को हाथ हिलाएं जिसे आप पहचानते हैं।",
        },
        {
          "id": "S6",
          "title": "दरवाजा पकड़ना",
          "desc": "अपने पीछे आने वाले व्यक्ति के लिए दरवाजा खुला रखें।",
        },
        {
          "id": "S7",
          "title": "सिर हिलाना",
          "desc":
              "पास से गुजरते समय किसी सहकर्मी को मित्रतापूर्वक सिर हिलाकर अभिवादन करें।",
        },
        {
          "id": "S8",
          "title": "दर्पण अभ्यास",
          "desc":
              "1 मिनट के लिए दर्पण में अपनी 'आत्मविश्वासी मुस्कान' का अभ्यास करें।",
        },
        {
          "id": "S9",
          "title": "संक्षिप्त नज़र",
          "desc":
              "किसी को 2 सेकंड के लिए देखें, फिर मुस्कुराएं और नज़र हटा लें।",
        },
        {
          "id": "S10",
          "title": "शांत प्रशंसा",
          "desc": "किसी के सोशल मीडिया पोस्ट पर एक अच्छा कमेंट लिखें।",
        },
        {
          "id": "S11",
          "title": "स्थान साझा करना",
          "desc":
              "सार्वजनिक क्षेत्र में किसी के बगल में बैठें और तुरंत नज़र न हटाएं।",
        },
        {
          "id": "S12",
          "title": "साधारण स्वीकृति",
          "desc":
              "गलियारे में किसी के पास से गुजरते समय विनम्रता से 'क्षमा करें' कहें।",
        },
        {
          "id": "S13",
          "title": "गर्मजोशी भरा अभिवादन",
          "desc": "किसी डिलीवरी ड्राइवर या कूरियर को 'नमस्ते' कहें।",
        },
        {
          "id": "S14",
          "title": "छोटी सी लहर",
          "desc":
              "किसी बच्चे या पालतू जानवर (मालिक की अनुमति से) को हाथ हिलाएं।",
        },
        {
          "id": "S15",
          "title": "कोमल मुस्कान",
          "desc": "आज तीन अलग-अलग लोगों को देखकर मुस्कुराएं।",
        },
        {
          "id": "S16",
          "title": "आई-कॉन्टैक्ट चुनौती",
          "desc":
              "कैशियर के साथ तब तक नज़रें मिलाएं जब तक कि वह पहले नज़र न हटा ले।",
        },
        {
          "id": "S17",
          "title": "कोमल सांस",
          "desc": "आज किसी सामाजिक स्थान पर जाने से पहले 3 गहरी सांसें लें।",
        },
        {
          "id": "S18",
          "title": "उपस्थिति",
          "desc":
              "भीड़भाड़ वाले क्षेत्र में 5 मिनट तक बिना फोन देखे खड़े रहें।",
        },
        {
          "id": "S19",
          "title": "अनौपचारिक सिर हिलाना",
          "desc":
              "किसी अजनबी को सिर हिलाकर अभिवादन करें जो आपसे नज़रें मिलाता है।",
        },
        {
          "id": "S20",
          "title": "कोमल आवाज़",
          "desc": "दुकान से बाहर निकलते समय किसी को 'आपका दिन शुभ हो' कहें।",
        },
      ],
      "Sprout": [
        {
          "id": "SP1",
          "title": "तारीफ",
          "desc": "किसी सहकर्मी या सहपाठी की सच्ची तारीफ करें।",
        },
        {
          "id": "SP2",
          "title": "प्रश्न",
          "desc": "किसी अजनबी से समय या दिशा-निर्देश पूछें।",
        },
        {
          "id": "SP3",
          "title": "छोटी बातचीत",
          "desc": "किसी से पूछें 'आपका दिन कैसा जा रहा है?' और उत्तर सुनें।",
        },
        {
          "id": "SP4",
          "title": "अनुरोध",
          "desc":
              "किसी विशिष्ट वस्तु को खोजने में मदद के लिए स्टोर कर्मचारी से पूछें।",
        },
        {
          "id": "SP5",
          "title": "ऑर्डर",
          "desc": "एक पेय या भोजन ऑर्डर करें और स्टाफ से पूछें कि वे कैसे हैं।",
        },
        {
          "id": "SP6",
          "title": "अभिवादन",
          "desc": "अपने क्षेत्र में किसी नए व्यक्ति से अपना परिचय दें।",
        },
        {
          "id": "SP7",
          "title": "मौसम की बात",
          "desc":
              "लाइन में प्रतीक्षा करते समय किसी से मौसम के बारे में बात करें।",
        },
        {
          "id": "SP8",
          "title": "साधारण पूछताछ",
          "desc": "किसी सहकर्मी से पूछें 'आपने वीकेंड पर क्या किया?'",
        },
        {
          "id": "SP9",
          "title": "मदद की पेशकश",
          "desc":
              "किसी से पूछें 'क्या आपको इसमें मदद चाहिए?' यदि वे संघर्ष करते दिखें।",
        },
        {
          "id": "SP10",
          "title": "राय",
          "desc":
              "किसी मित्र से किसी छोटी वस्तु के बारे में पूछें 'आप इसके बारे में क्या सोचते हैं?'",
        },
        {
          "id": "SP11",
          "title": "पुष्टि",
          "desc":
              "किसी अजनबी से विवरण की पुष्टि करें (जैसे, 'क्या यह सही लाइन है?').",
        },
        {
          "id": "SP12",
          "title": "साझा स्थान",
          "desc":
              "वातावरण के बारे में एक छोटी टिप्पणी करें (जैसे, 'आज बहुत भीड़ है').",
        },
        {
          "id": "SP13",
          "title": "छोटी मदद",
          "desc": "किसी से टेबल पर कुछ पास करने के लिए कहें (जैसे नैपकिन).",
        },
        {
          "id": "SP14",
          "title": "गर्म प्रतिक्रिया",
          "desc": "जाने से पहले वेटर को बताएं कि खाना बहुत अच्छा था।",
        },
        {
          "id": "SP15",
          "title": "अनौपचारिक हाल-चाल",
          "desc":
              "किसी ऐसे व्यक्ति को 'आप कैसे हैं?' मैसेज भेजें जिससे आपने एक महीने से बात नहीं की है।",
        },
        {
          "id": "SP16",
          "title": "खुला प्रश्न",
          "desc": "किसी से पूछें 'इस शहर में आपकी पसंदीदा जगह कौन सी है?'",
        },
        {
          "id": "SP17",
          "title": "सबसे छोटा जोखिम",
          "desc":
              "किसी अजनबी से पूछें कि क्या वे जानते हैं कि निकटतम शौचालय कहाँ है।",
        },
        {
          "id": "SP18",
          "title": "वस्तु की प्रशंसा",
          "desc": "किसी को बताएं कि आपको उनके जूते/बैग/एक्सेसरी पसंद आई।",
        },
        {
          "id": "SP19",
          "title": "विनम्र ठहराव",
          "desc":
              "किसी के पूरी तरह से बोलना समाप्त करने तक प्रतीक्षा करें और फिर उत्तर दें।",
        },
        {
          "id": "SP20",
          "title": "मैत्रीपूर्ण लहर",
          "desc":
              "किसी ऐसे व्यक्ति को हाथ हिलाकर 'बाय' कहें जिसके साथ आपने अभी संक्षिप्त बातचीत की हो।",
        },
      ],
      "Leaf": [
        {
          "id": "L1",
          "title": "राय जानना",
          "desc":
              "किसी से किसी किताब, फिल्म या गाने के बारे में उनकी राय पूछें।",
        },
        {
          "id": "L2",
          "title": "विवरण",
          "desc":
              "जब कोई अपने बारे में कुछ बताए, तो उसके बाद एक फॉलो-अप प्रश्न पूछें।",
        },
        {
          "id": "L3",
          "title": "सिफारिश",
          "desc":
              "किसी अजनबी से पास में खाने के लिए एक अच्छी जगह की सिफारिश मांगें।",
        },
        {
          "id": "L4",
          "title": "जुड़ाव",
          "desc":
              "किसी के साथ एक समान रुचि खोजें और उसके बारे में 2 मिनट तक बात करें।",
        },
        {
          "id": "L5",
          "title": "मददगार हाथ",
          "desc": "किसी को छोटे काम (जैसे बैग उठाने) में मदद की पेशकश करें।",
        },
        {
          "id": "L6",
          "title": "सामाजिक अवलोकन",
          "desc": "अपने आसपास हो रही किसी चीज़ के आधार पर बातचीत शुरू करें।",
        },
        {
          "id": "L7",
          "title": "खुला प्रश्न",
          "desc": "किसी से पूछें 'आप इस काम में कैसे आए?'",
        },
        {
          "id": "L8",
          "title": "सक्रिय श्रोता",
          "desc":
              "किसी को 3 मिनट तक बिना टोके सुनें, फिर उन्होंने जो कहा उसका सारांश बताएं।",
        },
        {
          "id": "L9",
          "title": "साझा हंसी",
          "desc": "एक छोटे समूह को एक छोटी, मजेदार कहानी या चुटकुला सुनाएं।",
        },
        {
          "id": "L10",
          "title": "जिज्ञासा",
          "desc":
              "किसी से पूछें कि वे कहाँ से हैं और उन्हें उस जगह के बारे में क्या पसंद है।",
        },
        {
          "id": "L11",
          "title": "सच्ची रुचि",
          "desc": "किसी सहकर्मी से उनके काम के बाहर के शौक के बारे में पूछें।",
        },
        {
          "id": "L12",
          "title": "कोमल सलाह",
          "desc": "किसी को उस चीज़ पर उपयोगी सुझाव दें जिसमें आप अच्छे हैं।",
        },
        {
          "id": "L13",
          "title": "समूह सहमति",
          "desc": "एक छोटे समूह की चर्चा में किसी की बात से सहमत हों।",
        },
        {
          "id": "L14",
          "title": "अनौपचारिक निमंत्रण",
          "desc":
              "किसी से पूछें 'क्या आप हमारे साथ दोपहर के भोजन पर चलना चाहेंगे?'",
        },
        {
          "id": "L15",
          "title": "ईमानदार प्रतिबिंब",
          "desc":
              "किसी को बताएं 'जब आपने X किया तो मैंने वास्तव में उसकी सराहना की' और समझाएं क्यों।",
        },
        {
          "id": "L16",
          "title": "जिज्ञासा अंतर",
          "desc":
              "किसी से पूछें 'मैं हमेशा सोचता था, X वास्तव में कैसे काम करता है?'",
        },
        {
          "id": "L17",
          "title": "छोटा समूह नेतृत्व",
          "desc":
              "एक ऐसा प्रश्न पूछें जिसके उत्तर के लिए समूह के 2 या 3 लोगों की आवश्यकता हो।",
        },
        {
          "id": "L18",
          "title": "सच्ची तारीफ",
          "desc":
              "किसी के व्यक्तित्व गुण की तारीफ करें (जैसे, 'आप एक बहुत अच्छे श्रोता हैं').",
        },
        {
          "id": "L19",
          "title": "साझा अनुभव",
          "desc": "बातचीत के दौरान कहें 'मैं भी उसी स्थिति में रहा हूँ'।",
        },
        {
          "id": "L20",
          "title": "अर्थपूर्ण ठहराव",
          "desc": "बातचीत में सन्नाटे को आने दें, उसे भरने की जल्दबाजी न करें।",
        },
      ],
      "Stem": [
        {
          "id": "ST1",
          "title": "साहसी शुरुआत",
          "desc":
              "किसी ऐसे व्यक्ति से बातचीत शुरू करें जिसे आप अच्छी तरह नहीं जानते।",
        },
        {
          "id": "ST2",
          "title": "ईमानदार साझाकरण",
          "desc": "समूह सेटिंग में एक छोटी व्यक्तिगत कहानी या राय साझा करें।",
        },
        {
          "id": "ST3",
          "title": "बहस",
          "desc": "विनम्रतापूर्वक किसी की राय से असहमत हों और समझाएं क्यों।",
        },
        {
          "id": "ST4",
          "title": "समूह प्रवेश",
          "desc": "एक समूह बातचीत में शामिल हों और एक विचारशील वाक्य जोड़ें।",
        },
        {
          "id": "ST5",
          "title": "विषय नेतृत्व",
          "desc": "एक सामाजिक समूह में बातचीत का एक नया विषय लाएं।",
        },
        {
          "id": "ST6",
          "title": "सार्वजनिक प्रश्न",
          "desc": "सार्वजनिक बैठक या कक्षा सेटिंग में एक प्रश्न पूछें।",
        },
        {
          "id": "ST7",
          "title": "साहसी अनुरोध",
          "desc":
              "किसी अजनबी से पूछें कि क्या आप कैफे या पार्क में उनके बगल में बैठ सकते हैं।",
        },
        {
          "id": "ST8",
          "title": "बातचीत का सेतु",
          "desc":
              "दो ऐसे लोगों का परिचय कराएं जो एक दूसरे को नहीं जानते और एक समानता खोजें।",
        },
        {
          "id": "ST9",
          "title": "मुखर आवश्यकता",
          "desc":
              "विनम्रतापूर्वक किसी से हटने या कुछ ऐसा करना बंद करने के लिए कहें जो आपको परेशान कर रहा है।",
        },
        {
          "id": "ST10",
          "title": "कहानीकार",
          "desc": "3 या अधिक लोगों के समूह को कहानी सुनाने में नेतृत्व करें।",
        },
        {
          "id": "ST11",
          "title": "खुली चुनौती",
          "desc":
              "समूह में एक आम राय को मित्रवत और सम्मानजनक तरीके से चुनौती दें।",
        },
        {
          "id": "ST12",
          "title": "सामाजिक पहल",
          "desc":
              "कमरे में प्रवेश करते समय हर किसी को 'नमस्ते' कहने वाले पहले व्यक्ति बनें।",
        },
        {
          "id": "ST13",
          "title": "सहानुभूतिपूर्ण श्रवण",
          "desc":
              "किसी को अपनी भड़ास निकालते हुए सुनें और एक सहायक प्रतिक्रिया दें।",
        },
        {
          "id": "ST14",
          "title": "सार्वजनिक प्रस्तुति",
          "desc": "एक सामाजिक सभा में अपने पसंदीदा विषय पर 1-2 मिनट तक बोलें।",
        },
        {
          "id": "ST15",
          "title": "कमजोरी साझा करना",
          "desc":
              "एक समूह में स्वीकार करें कि आप किसी चीज़ को लेकर घबराए हुए थे, और उस पर हंसें।",
        },
        {
          "id": "ST16",
          "title": "सीमा निर्धारित करना",
          "desc":
              "बिना अधिक स्पष्टीकरण दिए, विनम्रतापूर्वक एक निमंत्रण को अस्वीकार करें जिसे आप स्वीकार नहीं करना चाहते।",
        },
        {
          "id": "ST17",
          "title": "सक्रिय मध्यस्थ",
          "desc":
              "किसी असहमति में दो लोगों को बीच का रास्ता खोजने में मदद करें।",
        },
        {
          "id": "ST18",
          "title": "सार्वजनिक प्रशंसा",
          "desc":
              "समूह में किसी के प्रयास या उपलब्धि की सार्वजनिक रूप से प्रशंसा करें।",
        },
        {
          "id": "ST19",
          "title": "सीधा दृष्टिकोण",
          "desc":
              "किसी से सीधे किसी मदद या सलाह के लिए पूछें जिसकी आपको आवश्यकता है।",
        },
        {
          "id": "ST20",
          "title": "बातचीत का मोड़",
          "desc":
              "एक उबाऊ विषय से बातचीत को सहजता से एक दिलचस्प विषय की ओर मोड़ें।",
        },
      ],
      "Bloom": [
        {
          "id": "B1",
          "title": "उपहार",
          "desc":
              "किसी को एक छोटी सी मिठाई दें और कहें 'मुझे लगा कि आपको यह पसंद आएगा'।",
        },
        {
          "id": "B2",
          "title": "साहसी नेतृत्व",
          "desc":
              "लोगों के एक छोटे समूह को किसी योजना या घूमने की जगह का सुझाव दें।",
        },
        {
          "id": "B3",
          "title": "प्रशंसा",
          "desc":
              "किसी को विशेष रूप से बताएं कि आप उन्हें अपने जीवन में क्यों महत्व देते हैं।",
        },
        {
          "id": "B4",
          "title": "सामाजिक मेजबान",
          "desc":
              "कुछ लोगों के लिए एक छोटा गेट-टुगेदर या कॉफी डेट आयोजित करें।",
        },
        {
          "id": "B5",
          "title": "गहरी बातचीत",
          "desc":
              "किसी के साथ 15 मिनट से अधिक समय तक गहरी, सार्थक बातचीत करें।",
        },
        {
          "id": "B6",
          "title": "आत्मविश्वास का शिखर",
          "desc":
              "किसी ऐसे व्यक्ति के साथ बातचीत शुरू करें जिसे आप डरावना पाते हैं।",
        },
        {
          "id": "B7",
          "title": "सार्वजनिक टोस्ट",
          "desc":
              "समूह में किसी के लिए एक छोटा, सकारात्मक टोस्ट या प्रशंसा करें।",
        },
        {
          "id": "B8",
          "title": "सीमा निर्धारित करना",
          "desc":
              "किसी अनुरोध को दृढ़ता लेकिन विनम्रता से 'ना' कहें, बिना अधिक स्पष्टीकरण के।",
        },
        {
          "id": "B9",
          "title": "सीधा अनुरोध",
          "desc":
              "जिस व्यक्ति की आप प्रशंसा करते हैं, उससे 10 मिनट की बातचीत या मार्गदर्शन मांगें।",
        },
        {
          "id": "B10",
          "title": "भावनात्मक नेतृत्व",
          "desc":
              "किसी मित्र के साथ भावनाओं या मानसिक स्वास्थ्य के बारे में बातचीत शुरू करें।",
        },
        {
          "id": "B11",
          "title": "सामाजिक मध्यस्थ",
          "desc":
              "एक शांत बातचीत के माध्यम से दो लोगों को छोटा विवाद सुलझाने में मदद करें।",
        },
        {
          "id": "B12",
          "title": "साहसी तारीफ",
          "desc":
              "एक पूर्ण अजनबी को बताएं कि आप उनमें वास्तव में किस बात की प्रशंसा करते हैं।",
        },
        {
          "id": "B13",
          "title": "नेटवर्किंग मूव",
          "desc":
              "अपने क्षेत्र के किसी पेशेवर से अपना परिचय कराएं और सलाह मांगें।",
        },
        {
          "id": "B14",
          "title": "साहसी सच्चाई",
          "desc":
              "किसी को ऐसी सच्चाई बताएं जो कहना कठिन हो, लेकिन रिश्ते के लिए उपयोगी हो।",
        },
        {
          "id": "B15",
          "title": "पूर्ण प्रस्फुटन",
          "desc":
              "एक छोटा सामाजिक कार्यक्रम आयोजित करें और सुनिश्चित करें कि हर मेहमान का स्वागत महसूस हो।",
        },
        {
          "id": "B16",
          "title": "सार्वजनिक वक्ता",
          "desc":
              "किसी बैठक या कार्यक्रम के छोटे हिस्से को बोलने या निर्देशित करने के लिए स्वयंसेवा करें।",
        },
        {
          "id": "B17",
          "title": "कमजोरी का नेतृत्व",
          "desc":
              "किसी और को प्रोत्साहित करने के लिए एक संघर्ष साझा करें जिसे आपने पार किया हो।",
        },
        {
          "id": "B18",
          "title": "साहसी क्षमा",
          "desc":
              "किसी पुरानी गलती के लिए माफी मांगने के लिए बातचीत शुरू करें, भले ही वह बहुत समय पहले की हो।",
        },
        {
          "id": "B19",
          "title": "मेंटर",
          "desc":
              "किसी ऐसे व्यक्ति की मदद करें जो किसी कौशल में आपसे कम अनुभवी है।",
        },
        {
          "id": "B20",
          "title": "सामाजिक वास्तुकार",
          "desc":
              "दोस्तों के समूह के लिए एक नई सामाजिक परंपरा या एक आवर्ती मुलाकात बनाएं।",
        },
      ],
    },
  };

  static List<Map<String, String>> getTasks(String level, String lang) {
    return translations[lang]?[level] ?? translations['en']![level]!;
  }
}

// --- 1. SPLASH SCREEN ---
class SplashScreen extends StatefulWidget {
  final Function(String) onUserFound;
  final VoidCallback onNoUser;
  const SplashScreen({
    super.key,
    required this.onUserFound,
    required this.onNoUser,
  });

  @override
  State<SplashScreen> createState() => _SplashScreenState();
}

class _SplashScreenState extends State<SplashScreen> {
  @override
  void initState() {
    super.initState();
    _checkUserStatus();
  }

  void _checkUserStatus() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    String? userName = prefs.getString('userName');
    Future.delayed(const Duration(seconds: 3), () {
      if (userName != null) {
        widget.onUserFound(userName);
        Navigator.pushReplacement(
          context,
          MaterialPageRoute(
            builder: (context) => LevelMapScreen(userName: userName),
          ),
        );
      } else {
        widget.onNoUser();
        Navigator.pushReplacement(
          context,
          MaterialPageRoute(builder: (context) => const WelcomeScreen()),
        );
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: GlobalSettings.themeColor.value,
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(Icons.wb_sunny, size: 100, color: Colors.white),
            const SizedBox(height: 20),
            Text(
              "BLOOM",
              style: TextStyle(
                fontSize: 40,
                fontWeight: FontWeight.bold,
                color: Theme.of(context).colorScheme.surfaceVariant,
                letterSpacing: 4,
              ),
            ),
            const SizedBox(height: 10),
            const CircularProgressIndicator(color: Colors.white),
          ],
        ),
      ),
    );
  }
}

// --- 2. WELCOME SCREEN ---
class WelcomeScreen extends StatefulWidget {
  const WelcomeScreen({super.key});

  @override
  State<WelcomeScreen> createState() => _WelcomeScreenState();
}

class _WelcomeScreenState extends State<WelcomeScreen> {
  final TextEditingController _nameController = TextEditingController();

  void _handleEntrance(String name) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setString('userName', name);
    if (!mounted) return;
    Navigator.pushReplacement(
      context,
      MaterialPageRoute(builder: (context) => LevelMapScreen(userName: name)),
    );
  }

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder(
      valueListenable: GlobalSettings.language,
      builder: (context, lang, _) {
        return Scaffold(
          backgroundColor: Theme.of(context).colorScheme.surface,
          body: Padding(
            padding: const EdgeInsets.all(30.0),
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                Text(
                  AppTexts.get('welcome', lang),
                  style: TextStyle(
                    fontSize: 32,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 10),
                Text(
                  AppTexts.get('subtitle', lang),
                  style: const TextStyle(fontSize: 16, color: Colors.grey),
                  textAlign: TextAlign.center,
                ),
                const SizedBox(height: 40),
                TextField(
                  controller: _nameController,
                  // This makes the name the user types bold and high-contrast
                  style: TextStyle(
                    color: Theme.of(context).colorScheme.onSurface,
                    fontSize: 16,
                    fontWeight: FontWeight.w500,
                  ),
                  decoration: InputDecoration(
                    // This makes the "Enter your name..." hint soft and muted
                    hintStyle: TextStyle(
                      color: Theme.of(
                        context,
                      ).colorScheme.onSurfaceVariant.withOpacity(0.6),
                      fontSize: 15,
                    ),
                    hintText: "Enter your name or nickname",
                    // This makes the box color react to Light/Dark mode
                    fillColor: Theme.of(context).colorScheme.surfaceVariant,
                    filled: true,
                    border: OutlineInputBorder(
                      borderRadius: BorderRadius.circular(15),
                      borderSide: BorderSide.none,
                    ),
                  ),
                ),

                const SizedBox(height: 20),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: () {
                      String name = _nameController.text.isEmpty
                          ? "Brave Soul"
                          : _nameController.text;
                      _handleEntrance(name);
                    },
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.teal,
                      foregroundColor: Colors.white,
                      padding: const EdgeInsets.symmetric(vertical: 15),
                      shape: RoundedRectangleBorder(
                        borderRadius: BorderRadius.circular(15),
                      ),
                    ),
                    child: Text(
                      AppTexts.get('start', lang),
                      style: const TextStyle(fontSize: 18),
                    ),
                  ),
                ),
                TextButton(
                  onPressed: () => _handleEntrance("Guest"),
                  child: Text(
                    AppTexts.get('guest', lang),
                    style: const TextStyle(color: Colors.teal),
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}

// --- 3. LEVEL MAP ---
class LevelMapScreen extends StatefulWidget {
  final String userName;
  const LevelMapScreen({super.key, required this.userName});

  @override
  State<LevelMapScreen> createState() => _LevelMapScreenState();
}

class _LevelMapScreenState extends State<LevelMapScreen> {
  int _totalCompleted = 0;
  int _currentStreak = 0;

  @override
  void initState() {
    super.initState();
    _loadUserData();
  }

  void _loadUserData() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    setState(() {
      _totalCompleted = prefs.getInt('totalCompleted') ?? 0;
      _currentStreak = prefs.getInt('currentStreak') ?? 0;
    });
  }

  void _updateProgressAndStreak(String levelName) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    Map<String, double> weights = {
      "Seedling": 1.0,
      "Sprout": 2.0,
      "Leaf": 4.0,
      "Stem": 7.0,
      "Bloom": 12.0,
    };
    double currentScore = prefs.getDouble('confidenceScore') ?? 0.0;
    double addedPoints = weights[levelName] ?? 1.0;
    double newScore = currentScore + addedPoints;
    await prefs.setDouble('confidenceScore', newScore);

    String today = DateTime.now().toString().split(' ')[0];
    String? lastDate = prefs.getString('lastCompletedDate');
    int streak = prefs.getInt('currentStreak') ?? 0;
    if (lastDate != today) {
      DateTime now = DateTime.now();
      DateTime last = DateTime.parse(lastDate ?? "1970-01-01");
      if (now.difference(last).inDays == 1)
        streak++;
      else
        streak = 1;
      await prefs.setInt('currentStreak', streak);
      await prefs.setString('lastCompletedDate', today);
      int bestStreak = prefs.getInt('bestStreak') ?? 0;
      if (streak > bestStreak) await prefs.setInt('bestStreak', streak);
    }

    setState(() {
      _totalCompleted = newScore.toInt();
      _currentStreak = prefs.getInt('currentStreak') ?? 0;
    });
  }

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder(
      valueListenable: GlobalSettings.language,
      builder: (context, lang, _) {
        double progress = (_totalCompleted / 100).clamp(0.0, 1.0);
        return Scaffold(
          backgroundColor: Theme.of(context).colorScheme.surface,
          appBar: AppBar(
            title: Text("${AppTexts.get('hello', lang)}, ${widget.userName}!"),
            backgroundColor: Colors.transparent,
            elevation: 0,
            foregroundColor: Theme.of(context).colorScheme.primary,
            actions: [
              GestureDetector(
                onTap: () => Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => const StreakScreen()),
                ),
                child: Container(
                  margin: const EdgeInsets.only(right: 15),
                  padding: const EdgeInsets.symmetric(
                    horizontal: 12,
                    vertical: 6,
                  ),
                  decoration: BoxDecoration(
                    color: Colors.orangeAccent,
                    borderRadius: BorderRadius.circular(20),
                  ),
                  child: Row(
                    children: [
                      const Icon(
                        Icons.fireplace,
                        color: Colors.white,
                        size: 16,
                      ),
                      const SizedBox(width: 4),
                      Text(
                        "$_currentStreak",
                        style: const TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
              IconButton(
                icon: const Icon(Icons.person_outline),
                onPressed: () async {
                  await Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => const ProfileScreen(),
                    ),
                  );
                  _loadUserData();
                },
              ),
              IconButton(
                icon: const Icon(Icons.history),
                onPressed: () => Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const HistoryScreen(),
                  ),
                ),
              ),
              IconButton(
                icon: const Icon(Icons.help_outline),
                onPressed: () => Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => const FAQScreen()),
                ),
              ),
            ],
          ),
          body: Padding(
            padding: const EdgeInsets.all(20.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  AppTexts.get('progress', lang),
                  style: TextStyle(
                    fontSize: 20,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
                ),
                const SizedBox(height: 10),
                ClipRRect(
                  borderRadius: BorderRadius.circular(10),
                  child: LinearProgressIndicator(
                    value: progress,
                    minHeight: 15,
                    backgroundColor: Theme.of(
                      context,
                    ).colorScheme.primary.withOpacity(0.1),
                    valueColor: AlwaysStoppedAnimation<Color>(
                      Theme.of(context).colorScheme.primary,
                    ),
                  ),
                ),
                Text(
                  "${(progress * 100).toInt()}% Confident",
                  style: const TextStyle(fontSize: 12, color: Colors.grey),
                ),
                const SizedBox(height: 30),
                Text(
                  AppTexts.get('choose_level', lang),
                  style: TextStyle(
                    fontSize: 18,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
                ),
                const SizedBox(height: 20),
                Expanded(
                  child: ListView(
                    children: [
                      _levelCard(
                        context,
                        "Seedling",
                        Icons.grass,
                        Colors.green,
                      ),
                      const SizedBox(height: 15),
                      _levelCard(
                        context,
                        "Sprout",
                        Icons.eco,
                        Colors.lightGreen,
                      ),
                      const SizedBox(height: 15),
                      _levelCard(
                        context,
                        "Leaf",
                        Icons.opacity,
                        Colors.greenAccent,
                      ),
                      const SizedBox(height: 15),
                      _levelCard(context, "Stem", Icons.stadium, Colors.lime),
                      const SizedBox(height: 15),
                      _levelCard(
                        context,
                        "Bloom",
                        Icons.local_florist,
                        Colors.teal,
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }

  Widget _levelCard(
    BuildContext context,
    String levelName,
    IconData icon,
    Color color,
  ) {
    return InkWell(
      onTap: () async {
        final result = await Navigator.push(
          context,
          MaterialPageRoute(
            builder: (context) =>
                TaskScreen(userName: widget.userName, levelName: levelName),
          ),
        );
        if (result == true) _updateProgressAndStreak(levelName);
      },
      child: Container(
        padding: const EdgeInsets.all(20),
        decoration: BoxDecoration(
          color: Theme.of(context).colorScheme.surfaceVariant,
          borderRadius: BorderRadius.circular(20),
          boxShadow: [
            BoxShadow(
              color: Colors.black.withOpacity(0.05),
              blurRadius: 10,
              offset: const Offset(0, 5),
            ),
          ],
        ),
        child: Row(
          children: [
            CircleAvatar(
              radius: 25,
              backgroundColor: color.withOpacity(0.2),
              child: Icon(icon, color: color, size: 25),
            ),
            const SizedBox(width: 15),
            Text(
              levelName,
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            const Spacer(),
            const Icon(Icons.arrow_forward_ios, size: 14, color: Colors.grey),
          ],
        ),
      ),
    );
  }
}

// --- 4. STREAK SCREEN ---
class StreakScreen extends StatelessWidget {
  const StreakScreen({super.key});

  Future<Map<String, int>> _loadStreaks() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    return {
      "current": prefs.getInt('currentStreak') ?? 0,
      "best": prefs.getInt('bestStreak') ?? 0,
    };
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.surface,
      appBar: AppBar(
        title: const Text("My Streak"),
        backgroundColor: Colors.transparent,
        elevation: 0,
        foregroundColor: Theme.of(context).colorScheme.primary,
      ),
      body: FutureBuilder<Map<String, int>>(
        future: _loadStreaks(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting)
            return const Center(child: CircularProgressIndicator());
          int current = snapshot.data?['current'] ?? 0;
          int best = snapshot.data?['best'] ?? 0;
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: [
                const Icon(Icons.fireplace, size: 120, color: Colors.orange),
                const SizedBox(height: 20),
                const Text(
                  "Current Streak",
                  style: TextStyle(fontSize: 20, color: Colors.grey),
                ),
                Text(
                  "$current Days",
                  style: TextStyle(
                    fontSize: 48,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
                ),
                const SizedBox(height: 40),
                Container(
                  padding: const EdgeInsets.all(20),
                  decoration: BoxDecoration(
                    color: Theme.of(context).colorScheme.surfaceVariant,
                    borderRadius: BorderRadius.circular(20),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black.withOpacity(0.05),
                        blurRadius: 10,
                      ),
                    ],
                  ),
                  child: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      const Icon(Icons.emoji_events, color: Colors.amber),
                      const SizedBox(width: 10),
                      Text(
                        "Best Streak: $best Days",
                        style: const TextStyle(
                          fontSize: 18,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          );
        },
      ),
    );
  }
}

// --- 5. PROFILE SCREEN ---
class ProfileScreen extends StatefulWidget {
  const ProfileScreen({super.key});

  @override
  State<ProfileScreen> createState() => _ProfileScreenState();
}

class _ProfileScreenState extends State<ProfileScreen> {
  String _currentName = "";
  String _currentLang = 'en';
  Color _currentThemeColor = Colors.teal;
  final TextEditingController _editController = TextEditingController();

  final List<Map<String, dynamic>> themes = [
    {"name": "Classic Teal", "color": Colors.teal},
    {"name": "Ocean Blue", "color": Colors.blue},
    {"name": "Forest Green", "color": Colors.green},
    {"name": "Sunset Gold", "color": Colors.orange},
    {"name": "Midnight Purple", "color": Colors.deepPurple},
    {"name": "Stone Gray", "color": Colors.grey},
  ];

  final List<Map<String, String>> languages = [
    {"code": "en", "name": "English"},
    {"code": "es", "name": "Español"},
    {"code": "fr", "name": "Français"},
    {"code": "hi", "name": "हिन्दी"},
  ];

  @override
  void initState() {
    super.initState();
    _loadData();
  }

  void _loadData() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    setState(() {
      _currentName = prefs.getString('userName') ?? "Brave Soul";
      _currentLang = prefs.getString('language') ?? 'en';
      _currentThemeColor = Color(
        prefs.getInt('themeColor') ?? Colors.teal.value,
      );
      _editController.text = _currentName;
    });
  }

  void _updateName() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.setString('userName', _editController.text);
    setState(() => _currentName = _editController.text);
    ScaffoldMessenger.of(
      context,
    ).showSnackBar(const SnackBar(content: Text("Profile updated!")));
  }

  void _updateLanguage(String code) async {
    await GlobalSettings.saveLanguage(code);
    setState(() => _currentLang = code);
  }

  void _updateTheme(Color color) async {
    await GlobalSettings.saveTheme(color);
    setState(() => _currentThemeColor = color);
  }

  void _resetAllData() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    await prefs.clear();
    if (!mounted) return;
    Navigator.pushAndRemoveUntil(
      context,
      MaterialPageRoute(builder: (context) => const WelcomeScreen()),
      (route) => false,
    );
  }

  Widget _modeButton(String label, ThemeMode mode) {
    bool isSelected = GlobalSettings.themeMode.value == mode;
    return ElevatedButton(
      onPressed: () async {
        await GlobalSettings.saveThemeMode(mode);
        setState(() {}); // Refresh UI
      },
      style: ElevatedButton.styleFrom(
        backgroundColor: isSelected ? Colors.teal : Colors.white,
        foregroundColor: isSelected ? Colors.white : Colors.teal,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10)),
      ),
      child: Text(label),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.surface,
      appBar: AppBar(
        title: const Text("My Profile"),
        backgroundColor: Colors.transparent,
        elevation: 0,
        foregroundColor: Theme.of(context).colorScheme.primary,
      ),
      body: SingleChildScrollView(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            children: [
              CircleAvatar(
                radius: 50,
                backgroundColor: Theme.of(context).colorScheme.primary,
                child: Icon(Icons.person, size: 50, color: Colors.white),
              ),
              const SizedBox(height: 20),
              Text(
                _currentName,
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                  color: Theme.of(context).colorScheme.primary,
                ),
              ),
              const Text(
                "Confident Grower",
                style: TextStyle(color: Colors.grey),
              ),
              const SizedBox(height: 40),
              _buildSectionCard([
                TextField(
                  controller: _editController,
                  decoration: const InputDecoration(
                    labelText: "Update Name/Title",
                    border: OutlineInputBorder(),
                  ),
                ),
                const SizedBox(height: 20),
                SizedBox(
                  width: double.infinity,
                  child: ElevatedButton(
                    onPressed: _updateName,
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Theme.of(context).colorScheme.primary,
                      foregroundColor: Theme.of(context).colorScheme.surface,
                    ),
                    child: const Text("Save Changes"),
                  ),
                ),
              ]),
              const SizedBox(height: 20),
              _buildSectionCard([
                const Text(
                  "App Theme",
                  style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
                ),
                const SizedBox(height: 10),
                Wrap(
                  spacing: 10,
                  children: themes
                      .map(
                        (t) => GestureDetector(
                          onTap: () => _updateTheme(t['color'] as Color),
                          child: CircleAvatar(
                            radius: 15,
                            backgroundColor: t['color'] as Color,
                            child: _currentThemeColor == t['color']
                                ? const Icon(
                                    Icons.check,
                                    size: 15,
                                    color: Colors.white,
                                  )
                                : null,
                          ),
                        ),
                      )
                      .toList(),
                ),
                const SizedBox(height: 20),
              ]),
              const SizedBox(height: 20),
              _buildSectionCard([
                const Text(
                  "Language",
                  style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
                ),
                const SizedBox(height: 10),
                ...languages
                    .map(
                      (l) => ListTile(
                        title: Text(l['name']!),
                        leading: Radio(
                          value: l['code'],
                          groupValue: _currentLang,
                          onChanged: (val) => _updateLanguage(val as String),
                        ),
                      ),
                    )
                    .toList(),
              ]),
              const SizedBox(height: 60),
              // APPEARANCE MODE SECTION
              _buildSectionCard([
                const Text(
                  "Appearance Mode",
                  style: TextStyle(fontWeight: FontWeight.bold, fontSize: 16),
                ),
                const SizedBox(height: 10),
                Row(
                  mainAxisAlignment: MainAxisAlignment.spaceAround,
                  children: [
                    _modeButton("Light", ThemeMode.light),
                    _modeButton("Dark", ThemeMode.dark),
                  ],
                ),
              ]),
              const SizedBox(height: 20),
              TextButton(
                onPressed: () {
                  showDialog(
                    context: context,
                    builder: (context) => AlertDialog(
                      title: const Text("Reset Data?"),
                      content: const Text(
                        "This will delete all progress. Are you sure?",
                      ),
                      actions: [
                        TextButton(
                          onPressed: () => Navigator.pop(context),
                          child: const Text("Cancel"),
                        ),
                        TextButton(
                          onPressed: _resetAllData,
                          child: const Text(
                            "Reset",
                            style: TextStyle(color: Colors.red),
                          ),
                        ),
                      ],
                    ),
                  );
                },
                child: const Text(
                  "Reset All Progress",
                  style: TextStyle(color: Colors.red),
                ),
              ),
              const SizedBox(height: 40),
            ],
          ),
        ),
      ),
    );
  }

  Widget _buildSectionCard(List<Widget> children) {
    return Container(
      padding: const EdgeInsets.all(20),
      decoration: BoxDecoration(
        color: Theme.of(context).colorScheme.surfaceVariant,
        borderRadius: BorderRadius.circular(20),
        boxShadow: [
          BoxShadow(
            color: Colors.black.withOpacity(0.05),
            blurRadius: 10,
            offset: const Offset(0, 5),
          ),
        ],
      ),
      child: Column(children: children),
    );
  }
}

// --- 6. TASK SCREEN ---
class TaskScreen extends StatefulWidget {
  final String userName;
  final String levelName;
  const TaskScreen({
    super.key,
    required this.userName,
    required this.levelName,
  });

  @override
  State<TaskScreen> createState() => _TaskScreenState();
}

class _TaskScreenState extends State<TaskScreen> {
  late List<Map<String, String>> _sessionPool;
  Map<String, String>? _currentTask;
  int _completedCount = 0;
  bool _isCompleted = false;

  @override
  void initState() {
    super.initState();
    _initTaskPool();
  }

  void _initTaskPool() async {
    String lang = GlobalSettings.language.value;
    SharedPreferences prefs = await SharedPreferences.getInstance();
    List<String> completedIds = prefs.getStringList('completedTasks') ?? [];

    // Get tasks based on current language
    List<Map<String, String>> allLevelTasks = TaskLibrary.getTasks(
      widget.levelName,
      lang,
    );
    List<Map<String, String>> availableTasks = allLevelTasks
        .where((task) => !completedIds.contains(task['id']))
        .toList();

    setState(() {
      _sessionPool = availableTasks;
      _sessionPool.shuffle();
      _pickRandomTask();
    });
  }

  void _pickRandomTask() {
    setState(() {
      _currentTask = _sessionPool.isNotEmpty ? _sessionPool.first : null;
    });
  }

  void _completeTask() {
    if (_currentTask == null) return;
    setState(() => _isCompleted = true);
    Future.delayed(const Duration(milliseconds: 600), () {
      Navigator.push(
        context,
        MaterialPageRoute(
          builder: (context) => ReflectionScreen(
            taskTitle: _currentTask?['title'] ?? "Challenge",
            onReflectionDone: (anxiety, note) async {
              await _saveTaskCompletion(
                _currentTask?['id'] ?? "unknown",
                _currentTask?['title'] ?? "Unknown",
                _currentTask?['desc'] ?? "No desc",
                anxiety,
                note,
              );
              setState(() {
                _completedCount++;
                if (_sessionPool.isNotEmpty) _sessionPool.removeAt(0);
                _isCompleted = false;
                _pickRandomTask();
              });
              if (_completedCount % 10 == 0) _showMilestoneDialog();
            },
          ),
        ),
      );
    });
  }

  Future<void> _saveTaskCompletion(
    String id,
    String title,
    String desc,
    int anxiety,
    String note,
  ) async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    List<String> completedIds = prefs.getStringList('completedTasks') ?? [];
    completedIds.add(id);
    await prefs.setStringList('completedTasks', completedIds);
    String? historyJson = prefs.getString('history');
    List<dynamic> historyList = historyJson != null
        ? jsonDecode(historyJson)
        : [];
    historyList.add({
      "title": title,
      "desc": desc,
      "date": DateFormat('MMM d, yyyy - hh:mm a').format(DateTime.now()),
      "anxiety": anxiety,
      "note": note,
    });
    await prefs.setString('history', jsonEncode(historyList));
  }

  void _showMilestoneDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text(
          widget.levelName == "Bloom" ? "🌟 Full Bloom!" : "✨ You're Growing!",
        ),
        content: Text(
          widget.levelName == "Bloom"
              ? "You've reached the peak of confidence!"
              : "Ready for a harder level?",
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text("I'll stay here"),
          ),
          ElevatedButton(
            onPressed: () {
              Navigator.pop(context);
              Navigator.pop(context, true);
            },
            child: const Text("Explore Next Level"),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return ValueListenableBuilder(
      valueListenable: GlobalSettings.language,
      builder: (context, lang, _) {
        if (_currentTask == null) {
          return Scaffold(
            backgroundColor: Theme.of(context).colorScheme.surface,
            body: Center(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  const Icon(Icons.stars, size: 80, color: Colors.teal),
                  const SizedBox(height: 20),
                  Text(
                    "Stage Mastered!",
                    style: TextStyle(
                      fontSize: 24,
                      fontWeight: FontWeight.bold,
                      color: Theme.of(context).colorScheme.primary,
                    ),
                  ),
                  const SizedBox(height: 30),
                  ElevatedButton(
                    onPressed: () => Navigator.pop(context, true),
                    style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.teal,
                      foregroundColor: Colors.white,
                    ),
                    child: const Text("Return to Map"),
                  ),
                ],
              ),
            ),
          );
        }

        return Scaffold(
          backgroundColor: Theme.of(context).colorScheme.surface,
          appBar: AppBar(
            backgroundColor: Colors.transparent,
            elevation: 0,
            leading: BackButton(
              color: Theme.of(context).colorScheme.primary,
              onPressed: () {
                if (_completedCount > 0) {
                  Navigator.pop(context, true);
                } else {
                  Navigator.pop(context);
                }
              },
            ),
          ),
          body: Padding(
            padding: const EdgeInsets.all(24.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  "Go ${widget.userName}!",
                  style: TextStyle(
                    fontSize: 24,
                    fontWeight: FontWeight.bold,
                    color: Theme.of(context).colorScheme.primary,
                  ),
                ),
                Text(
                  "Stage: ${widget.levelName}",
                  style: const TextStyle(fontSize: 16, color: Colors.grey),
                ),
                const SizedBox(height: 40),
                Container(
                  width: double.infinity,
                  padding: const EdgeInsets.all(30),
                  decoration: BoxDecoration(
                    color: Theme.of(context).colorScheme.surfaceVariant,
                    borderRadius: BorderRadius.circular(30),
                    boxShadow: [
                      BoxShadow(
                        color: Colors.black.withOpacity(0.05),
                        blurRadius: 15,
                        offset: const Offset(0, 8),
                      ),
                    ],
                  ),
                  child: Column(
                    children: [
                      Icon(
                        _isCompleted
                            ? Icons.check_circle
                            : Icons.wb_sunny_outlined,
                        color: _isCompleted ? Colors.green : Colors.orange,
                        size: 50,
                      ),
                      const SizedBox(height: 20),
                      Text(
                        _currentTask?['title'] ?? "Challenge",
                        style: const TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                        textAlign: TextAlign.center,
                      ),
                      const SizedBox(height: 15),
                      Text(
                        _currentTask?['desc'] ?? "Description loading...",
                        style: TextStyle(
                          fontSize: 16,
                          color: Theme.of(context).colorScheme.onSurfaceVariant,
                        ),
                        textAlign: TextAlign.center,
                      ),
                      const SizedBox(height: 30),
                      Text(
                        "Tasks completed this session: $_completedCount",
                        style: TextStyle(
                          fontSize: 12,
                          color: Theme.of(context).colorScheme.onSurfaceVariant,
                        ),
                      ),
                      const SizedBox(height: 20),
                      SizedBox(
                        width: double.infinity,
                        child: ElevatedButton(
                          onPressed: _isCompleted ? null : _completeTask,
                          style: ElevatedButton.styleFrom(
                            backgroundColor: Theme.of(
                              context,
                            ).colorScheme.primary,
                            foregroundColor: Colors.white,
                            padding: const EdgeInsets.symmetric(vertical: 15),
                            shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.circular(15),
                            ),
                          ),
                          child: Text(
                            _isCompleted ? "Great job!" : "I did it!",
                            style: const TextStyle(fontSize: 18),
                          ),
                        ),
                      ),
                    ],
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}

// --- 6. REFLECTION SCREEN ---
class ReflectionScreen extends StatefulWidget {
  final String taskTitle;
  final Function(int anxiety, String note) onReflectionDone;
  const ReflectionScreen({
    super.key,
    required this.taskTitle,
    required this.onReflectionDone,
  });

  @override
  State<ReflectionScreen> createState() => _ReflectionScreenState();
}

class _ReflectionScreenState extends State<ReflectionScreen> {
  double _anxietyLevel = 5;
  final TextEditingController _noteController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    String lang = GlobalSettings.language.value;
    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.surface,
      appBar: AppBar(backgroundColor: Colors.transparent, elevation: 0),
      body: SingleChildScrollView(
        // <--- Add this wrapper
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text(
                AppTexts.get('reflect', lang),
                style: TextStyle(
                  fontSize: 24,
                  fontWeight: FontWeight.bold,
                  color: Theme.of(context).colorScheme.primary,
                ),
              ),
              Text(
                "Task: ${widget.taskTitle}",
                style: const TextStyle(fontSize: 16, color: Colors.grey),
              ),
              const SizedBox(height: 40),
              Text(
                AppTexts.get('anxiety_q', lang),
                style: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.w500,
                ),
              ),
              Slider(
                value: _anxietyLevel,
                min: 1,
                max: 10,
                divisions: 9,
                activeColor: Theme.of(context).colorScheme.primary,
                onChanged: (v) => setState(() => _anxietyLevel = v),
              ),
              Text(
                "Level: ${_anxietyLevel.toInt()}",
                textAlign: TextAlign.center,
                style: const TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                ),
              ),
              const SizedBox(height: 30),
              Text(
                AppTexts.get('what_happened', lang),
                style: const TextStyle(
                  fontSize: 16,
                  fontWeight: FontWeight.w500,
                ),
              ),
              const SizedBox(height: 10),
              TextField(
                controller: _noteController,
                maxLines: 4,
                // This controls the text the user TYPES
                style: TextStyle(
                  color: Theme.of(
                    context,
                  ).colorScheme.onSurface, // Highest contrast color
                  fontSize: 16,
                  fontWeight: FontWeight
                      .w500, // Makes the input text slightly bolder/stronger
                ),
                decoration: InputDecoration(
                  // This controls the "I felt nervous, but..." HINT text
                  hintStyle: TextStyle(
                    color: Theme.of(context).colorScheme.onSurfaceVariant
                        .withOpacity(0.6), // Make hint more muted
                    fontSize: 15,
                    fontWeight: FontWeight.normal, // Keep hint light
                  ),
                  hintText: "I felt nervous, but...",
                  fillColor: Theme.of(context).colorScheme.surfaceVariant,
                  filled: true,
                  border: OutlineInputBorder(
                    borderRadius: BorderRadius.circular(15),
                    borderSide: BorderSide.none,
                  ),
                ),
              ),

              const SizedBox(height: 30),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: () {
                    widget.onReflectionDone(
                      _anxietyLevel.toInt(),
                      _noteController.text,
                    );
                    Navigator.pop(context);
                  },
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Theme.of(context).colorScheme.primary,
                    foregroundColor: Colors.white,
                    padding: const EdgeInsets.symmetric(vertical: 15),
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(15),
                    ),
                  ),
                  child: Text(
                    AppTexts.get('finish', lang),
                    style: const TextStyle(fontSize: 18),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// --- 7. HISTORY SCREEN ---
class HistoryScreen extends StatelessWidget {
  const HistoryScreen({super.key});

  Future<List<dynamic>> _loadHistory() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    String? historyJson = prefs.getString('history');
    return historyJson != null ? jsonDecode(historyJson) : [];
  }

  void _showTaskDetail(BuildContext context, dynamic entry) {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
        title: Text(
          entry['title'] ?? "Task Detail",
          style: TextStyle(
            color: Theme.of(context).colorScheme.primary,
            fontWeight: FontWeight.bold,
          ),
        ),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            const Text(
              "The Challenge:",
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Text(
              entry['desc'] ?? "Description not available.",
              style: TextStyle(
                fontSize: 15,
                color: Theme.of(context).colorScheme.onSurface,
              ),
            ),
            const SizedBox(height: 20),
            const Text(
              "Your Experience:",
              style: TextStyle(fontWeight: FontWeight.bold),
            ),
            Text(
              "Anxiety Level: ${entry['anxiety']}/10",
              style: const TextStyle(fontSize: 14),
            ),
            const SizedBox(height: 5),
            Text(
              entry['note'] ?? "No notes added.",
              style: TextStyle(
                fontSize: 15,
                fontStyle: FontStyle.italic,
                color: Theme.of(context).colorScheme.onSurfaceVariant,
              ),
            ),
            const SizedBox(height: 20),
            Text(
              "Completed on: ${entry['date']}",
              style: const TextStyle(fontSize: 12, color: Colors.grey),
            ),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(context),
            child: const Text("Close", style: TextStyle(color: Colors.teal)),
          ),
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.surface,
      appBar: AppBar(
        title: const Text("My Growth Journey"),
        backgroundColor: Colors.transparent,
        elevation: 0,
        foregroundColor: Theme.of(context).colorScheme.primary,
      ),
      body: FutureBuilder<List<dynamic>>(
        future: _loadHistory(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting)
            return const Center(child: CircularProgressIndicator());
          if (!snapshot.hasData || snapshot.data!.isEmpty)
            return const Center(
              child: Text("No reflections yet. Start your journey!"),
            );
          List<dynamic> history = snapshot.data!;
          return ListView.builder(
            padding: const EdgeInsets.all(20),
            itemCount: history.length,
            itemBuilder: (context, index) {
              var entry = history.reversed.toList()[index];
              return InkWell(
                onTap: () => _showTaskDetail(context, entry),
                child: Card(
                  margin: const EdgeInsets.only(bottom: 15),
                  shape: RoundedRectangleBorder(
                    borderRadius: BorderRadius.circular(15),
                  ),
                  child: ListTile(
                    contentPadding: const EdgeInsets.all(15),
                    title: Text(
                      entry['title'] ?? "Unknown Task",
                      style: const TextStyle(fontWeight: FontWeight.bold),
                    ),
                    subtitle: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Wrap(
                          spacing: 10,
                          children: [
                            Row(
                              mainAxisSize: MainAxisSize.min,
                              children: [
                                Icon(
                                  Icons.calendar_today,
                                  size: 10,
                                  color: Theme.of(
                                    context,
                                  ).colorScheme.onSurfaceVariant,
                                ),
                                const SizedBox(width: 5),
                                Text(
                                  "${entry['date']}",
                                  style: TextStyle(
                                    fontSize: 12,
                                    color: Theme.of(
                                      context,
                                    ).colorScheme.onSurfaceVariant,
                                  ),
                                ),
                              ],
                            ),
                            Row(
                              mainAxisSize: MainAxisSize.min,
                              children: [
                                const Icon(
                                  Icons.favorite,
                                  size: 10,
                                  color: Colors.redAccent,
                                ),
                                const SizedBox(width: 5),
                                Text(
                                  "Anxiety: ${entry['anxiety']}/10",
                                  style: TextStyle(
                                    fontSize: 12,
                                    color: Theme.of(
                                      context,
                                    ).colorScheme.onSurfaceVariant,
                                  ),
                                ),
                              ],
                            ),
                          ],
                        ),
                        const SizedBox(height: 10),
                        Text(
                          entry['note'] ?? "No notes added.",
                          style: const TextStyle(
                            fontSize: 14,
                            fontStyle: FontStyle.italic,
                          ),
                          maxLines: 2,
                          overflow: TextOverflow.ellipsis,
                        ),
                      ],
                    ),
                    trailing: Icon(
                      Icons.chevron_right,
                      color: Theme.of(context).colorScheme.primary,
                    ),
                  ),
                ),
              );
            },
          );
        },
      ),
    );
  }
}

class FAQScreen extends StatelessWidget {
  const FAQScreen({super.key});

  @override
  Widget build(BuildContext context) {
    // Define the FAQ data
    final List<Map<String, String>> faqData = [
      {
        "q": "What is Bloom?",
        "a":
            "Bloom is a self-help tool designed to help people reduce social anxiety through a process called 'Graded Exposure.' By completing small, manageable social tasks, you train your brain to realize that social interactions are safe and manageable.",
      },
      {
        "q": "How do the levels work?",
        "a":
            "We start with 'Seedling' (very easy tasks) and move up to 'Bloom' (more challenging tasks). As you complete tasks, you earn confidence points. The higher the level, the more points you earn!",
      },
      {
        "q": "What is the Confidence Meter?",
        "a":
            "The progress bar on your home screen represents your overall confidence. It grows as you complete tasks. Be careful: if you stop practicing for several days, your confidence score may decay slightly, reminding you that confidence is a muscle that needs regular exercise!",
      },
      {
        "q": "What is a Streak?",
        "a":
            "A streak is a count of how many consecutive days you have completed at least one task. Consistency is the key to overcoming anxiety, so try to keep your flame burning!",
      },
      {
        "q": "Where is my data stored?",
        "a":
            "Your privacy is our priority. All your progress, history, and profile data are stored locally on your own device. Nothing is uploaded to a cloud server.",
      },
      {
        "q": "What do I do if the app crashes?",
        "a":
            "If the app behaves strangely, try restarting your phone. If you've updated the app, you might need to clear the app cache in your Android settings. If all else fails, you can use the 'Reset All Progress' option in your Profile.",
      },
      {
        "q": "Can I skip levels?",
        "a":
            "Yes! While we recommend the gradual path, you are free to choose any level from the map that feels appropriate for your current comfort level.",
      },
      {
        "q": "What if the task is very hard?",
        "a":
            "While we recommend you to try to accomplish the task, you can just go back to home screen, and re-enter to change the current task",
      },
    ];

    return Scaffold(
      backgroundColor: Theme.of(context).colorScheme.surface,
      appBar: AppBar(
        title: const Text("Help & FAQ"),
        backgroundColor: Colors.transparent,
        elevation: 0,
        foregroundColor: Theme.of(context).colorScheme.primary,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text(
              "Common Questions",
              style: TextStyle(
                fontSize: 22,
                fontWeight: FontWeight.bold,
                color: Theme.of(context).colorScheme.primary,
              ),
            ),
            const SizedBox(height: 20),
            Expanded(
              child: ListView.builder(
                itemCount: faqData.length,
                itemBuilder: (context, index) {
                  return Card(
                    margin: const EdgeInsets.only(bottom: 12),
                    shape: RoundedRectangleBorder(
                      borderRadius: BorderRadius.circular(15),
                    ),
                    color: Theme.of(context).colorScheme.surfaceVariant,
                    child: ExpansionTile(
                      title: Text(
                        faqData[index]['q']!,
                        style: const TextStyle(fontWeight: FontWeight.bold),
                      ),
                      children: [
                        Padding(
                          padding: const EdgeInsets.all(16.0),
                          child: Text(
                            faqData[index]['a']!,
                            style: TextStyle(
                              fontSize: 15,
                              color: Theme.of(
                                context,
                              ).colorScheme.onSurfaceVariant,
                            ),
                            textAlign: TextAlign.left,
                          ),
                        ),
                      ],
                    ),
                  );
                },
              ),
            ),
            const SizedBox(height: 20),
            Center(
              child: Text(
                "Still need help? Keep blooming! 🌸",
                style: TextStyle(
                  fontStyle: FontStyle.italic,
                  color: Colors.grey[600],
                ),
              ),
            ),
            const SizedBox(height: 20),
          ],
        ),
      ),
    );
  }
}
