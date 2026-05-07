import 'package:flutter/material.dart';
import 'dart:math';

void main() {
  runApp(const MaterialApp(
    debugShowCheckedModeBanner: false,
    home: SpacedRepetitionApp(),
  ));
}

class Flashcard {
  String question;
  String answer;
  double easeFactor;
  int interval;
  int repetitions;

  Flashcard({
    required this.question,
    required this.answer,
    this.easeFactor = 2.5,
    this.interval = 0,
    this.repetitions = 0,
  });
}

class SpacedRepetitionApp extends StatefulWidget {
  const SpacedRepetitionApp({super.key});

  @override
  State<SpacedRepetitionApp> createState() => _SpacedRepetitionAppState();
}

class _SpacedRepetitionAppState extends State<SpacedRepetitionApp> with TickerProviderStateMixin {
  int index = 0;
  bool _showingFront = true;
  
  // Expanded Question Deck
  final List<Flashcard> deck = [
    Flashcard(question: "What is Spaced Repetition?", answer: "An evidence-based learning technique using increasing intervals."),
    Flashcard(question: "What is SM-2?", answer: "An algorithm for calculating optimal review intervals based on performance."),
    Flashcard(question: "What is BS IT?", answer: "Bachelor of Science in Information Technology."),
    Flashcard(question: "What is Flutter's State?", answer: "Data that can be read synchronously when the widget is built and might change."),
    Flashcard(question: "SQL vs NoSQL?", answer: "SQL is relational/table-based; NoSQL is non-relational/document-based."),
    Flashcard(question: "What is a 'Widget' in Flutter?", answer: "The basic building block of a Flutter UI; everything is a widget."),
    Flashcard(question: "Difference between 'final' and 'const'?", answer: "'final' is a runtime constant; 'const' is a compile-time constant."),
    Flashcard(question: "What is an API?", answer: "Application Programming Interface: a way for two programs to talk to each other."),
    Flashcard(question: "What is Git?", answer: "A distributed version control system to track changes in source code."),
    Flashcard(question: "What is the purpose of 'pubspec.yaml'?", answer: "It manages the project's dependencies, assets, and metadata."),
    Flashcard(question: "What is Hot Reload?", answer: "A Flutter feature that injects updated code into the VM without restarting the app."),
  ];

  late AnimationController _flipController;
  late Animation<double> _flipAnimation;

  @override
  void initState() {
    super.initState();
    _flipController = AnimationController(vsync: this, duration: const Duration(milliseconds: 400));
    _flipAnimation = Tween<double>(begin: 0, end: pi).animate(
      CurvedAnimation(parent: _flipController, curve: Curves.easeInOut),
    );
  }

  void _toggleCard() {
    _showingFront ? _flipController.forward() : _flipController.reverse();
    setState(() => _showingFront = !_showingFront);
  }

  // Improved Navigation: Resets rotation immediately to avoid "ghosting" text
  void next() {
    setState(() {
      index = (index + 1) % deck.length;
      _resetFlip();
    });
  }

  void prev() {
    setState(() {
      index = (index - 1 < 0) ? deck.length - 1 : index - 1;
      _resetFlip();
    });
  }

  void _resetFlip() {
    _showingFront = true;
    _flipController.value = 0; // Instantly snap back to front for the new card
  }

  @override
  Widget build(BuildContext context) {
    Color primaryColor = _showingFront ? Colors.indigo : Colors.teal;

    return Scaffold(
      backgroundColor: Colors.grey[100],
      appBar: AppBar(
        title: const Text("Flashcards (SM2)", style: TextStyle(fontWeight: FontWeight.bold, color: Colors.white)),
        centerTitle: true,
        backgroundColor: primaryColor,
        elevation: 10,
      ),
      body: Column(
        children: [
          const SizedBox(height: 20),
          _buildStatsBar(primaryColor),
          const Spacer(),
          GestureDetector(
            onTap: _toggleCard,
            child: AnimatedBuilder(
              animation: _flipAnimation,
              builder: (context, child) {
                final angle = _flipAnimation.value;
                return Transform(
                  transform: Matrix4.identity()..setEntry(3, 2, 0.001)..rotateY(angle),
                  alignment: Alignment.center,
                  child: angle > pi / 2
                      ? Transform(
                          transform: Matrix4.identity()..rotateY(pi),
                          alignment: Alignment.center,
                          child: _cardSide(deck[index].answer, "ANSWER", Colors.teal),
                        )
                      : _cardSide(deck[index].question, "QUESTION", Colors.indigo),
                );
              },
            ),
          ),
          const Spacer(),
          Padding(
            padding: const EdgeInsets.symmetric(horizontal: 40, vertical: 30),
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                _navBtn(Icons.arrow_back, "Prev", prev, primaryColor),
                _navBtn(Icons.arrow_forward, "Next", next, primaryColor),
              ],
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {}, 
        backgroundColor: Colors.orange,
        child: const Icon(Icons.add, color: Colors.white),
      ),
    );
  }

  Widget _buildStatsBar(Color color) {
    return Container(
      margin: const EdgeInsets.symmetric(horizontal: 20),
      padding: const EdgeInsets.all(15),
      decoration: BoxDecoration(
        color: Colors.white,
        borderRadius: BorderRadius.circular(15),
        boxShadow: const [BoxShadow(color: Colors.black12, blurRadius: 5)],
      ),
      child: Row(
        mainAxisAlignment: MainAxisAlignment.spaceAround,
        children: [
          _statItem("Ease", "${deck[index].easeFactor}", color),
          _statItem("Interval", "${deck[index].interval}d", color),
          _statItem("Progress", "${index + 1}/${deck.length}", color),
        ],
      ),
    );
  }

  Widget _statItem(String label, String value, Color color) {
    return Column(
      children: [
        Text(label, style: TextStyle(fontSize: 12, color: Colors.grey[600])),
        Text(value, style: TextStyle(fontSize: 16, fontWeight: FontWeight.bold, color: color)),
      ],
    );
  }

  Widget _cardSide(String text, String label, Color color) {
    return Container(
      width: 320, height: 220,
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.circular(25),
        boxShadow: [BoxShadow(color: color.withOpacity(0.4), blurRadius: 15, offset: const Offset(0, 8))],
      ),
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(label, style: const TextStyle(color: Colors.white60, fontSize: 12, letterSpacing: 2)),
            const SizedBox(height: 20),
            Padding(
              padding: const EdgeInsets.symmetric(horizontal: 20),
              child: Text(text, textAlign: TextAlign.center, style: const TextStyle(color: Colors.white, fontSize: 18, fontWeight: FontWeight.bold)),
            ),
          ],
        ),
      ),
    );
  }

  Widget _navBtn(IconData icon, String label, VoidCallback action, Color color) {
    return ElevatedButton.icon(
      onPressed: action,
      icon: Icon(icon, size: 18),
      label: Text(label),
      style: ElevatedButton.styleFrom(
        backgroundColor: color,
        foregroundColor: Colors.white,
        padding: const EdgeInsets.symmetric(horizontal: 25, vertical: 12),
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
      ),
    );
  }
} 