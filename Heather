import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'dart:convert';
import 'dart:async';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final prefs = await SharedPreferences.getInstance();
  
  runApp(
    ChangeNotifierProvider(
      create: (_) => TimerAppState(prefs),
      child: const MultiTimerApp(),
    ),
  );
}

class MultiTimerApp extends StatelessWidget {
  const MultiTimerApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark().copyWith(
        scaffoldBackgroundColor: const Color(0xFF121214),
        cardColor: const Color(0xFF1A1A1E),
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
          brightness: Brightness.dark,
        ),
      ),
      home: const TimerDashboard(),
    );
  }
}

class TimerModel {
  final String id;
  final String label;
  final int totalSeconds;
  bool isRunning;
  DateTime? endTime; 
  int pausedRemainingSeconds; 

  TimerModel({
    required this.id,
    required this.label,
    required this.totalSeconds,
    this.isRunning = false,
    this.endTime,
    int? remainingSeconds,
  }) : pausedRemainingSeconds = remainingSeconds ?? totalSeconds;

  int get remainingSeconds {
    if (!isRunning) return pausedRemainingSeconds;
    if (endTime == null) return 0;
    final dynamicRemaining = endTime!.difference(DateTime.now()).inSeconds;
    return dynamicRemaining > 0 ? dynamicRemaining : 0;
  }

  double get progress => totalSeconds > 0 ? remainingSeconds / totalSeconds : 0.0;
  
  String get formattedTime {
    final secs = remainingSeconds;
    final minutes = (secs / 60).floor().toString().padLeft(2, '0');
    final seconds = (secs % 60).toString().padLeft(2, '0');
    return '$minutes:$seconds';
  }

  Map<String, dynamic> toJson() => {
    'id': id,
    'label': label,
    'totalSeconds': totalSeconds,
    'isRunning': isRunning,
    'endTime': endTime?.toIso8601String(),
    'pausedRemainingSeconds': pausedRemainingSeconds,
  };

  factory TimerModel.fromJson(Map<String, dynamic> json) => TimerModel(
    id: json['id'],
    label: json['label'],
    totalSeconds: json['totalSeconds'],
    isRunning: json['isRunning'],
    endTime: json['endTime'] != null ? DateTime.parse(json['endTime']) : null,
    remainingSeconds: json['pausedRemainingSeconds'],
  );
}

class TimerAppState extends ChangeNotifier {
  final SharedPreferences prefs;
  List<TimerModel> timers = [];
  Timer? _globalTicker;

  TimerAppState(this.prefs) {
    _loadTimersFromStorage();
    _startGlobalTicker();
  }

  void _loadTimersFromStorage() {
    final String? cachedData = prefs.getString('heather_timers');
    if (cachedData != null) {
      try {
        final List<dynamic> decoded = jsonDecode(cachedData);
        timers = decoded.map((item) => TimerModel.fromJson(item)).toList();
      } catch (e) {
        _loadDefaultTimers();
      }
    } else {
      _loadDefaultTimers();
    }
    notifyListeners();
  }

  void _loadDefaultTimers() {
    timers = [
      TimerModel(id: 't1', label: 'Task Alpha', totalSeconds: 1500),
      TimerModel(id: 't2', label: 'Beta Rest', totalSeconds: 300),
      TimerModel(id: 't3', label: 'Gamma Sprint', totalSeconds: 2700),
      TimerModel(id: 't4', label: 'Delta Cool', totalSeconds: 600),
    ];
    _saveTimersToStorage();
  }

  void _saveTimersToStorage() {
    final String encoded = jsonEncode(timers.map((t) => t.toJson()).toList());
    prefs.setString('heather_timers', encoded);
  }

  void _startGlobalTicker() {
    _globalTicker = Timer.periodic(const Duration(seconds: 1), (_) {
      bool activeRunning = false;
      bool stateChanged = false;
      
      for (var timer in timers) {
        if (timer.isRunning) {
          if (timer.remainingSeconds == 0) {
            timer.isRunning = false;
            timer.pausedRemainingSeconds = 0;
            timer.endTime = null;
            stateChanged = true;
          } else {
            activeRunning = true; 
          }
        }
      }
      
      if (stateChanged) {
        _saveTimersToStorage();
      }
      
      if (activeRunning || stateChanged) {
        notifyListeners();
      }
    });
  }

  void toggleTimer(String id) {
    final idx = timers.indexWhere((t) => t.id == id);
    if (idx != -1) {
      final timer = timers[idx];
      if (timer.isRunning) {
        timer.pausedRemainingSeconds = timer.remainingSeconds;
        timer.endTime = null;
        timer.isRunning = false;
      } else {
        if (timer.pausedRemainingSeconds == 0) {
          timer.pausedRemainingSeconds = timer.totalSeconds; 
        }
        timer.endTime = DateTime.now().add(Duration(seconds: timer.pausedRemainingSeconds));
        timer.isRunning = true;
      }
      _saveTimersToStorage();
      notifyListeners();
    }
  }

  void resetTimer(String id) {
    final idx = timers.indexWhere((t) => t.id == id);
    if (idx != -1) {
      timers[idx].isRunning = false;
      timers[idx].endTime = null;
      timers[idx].pausedRemainingSeconds = timers[idx].totalSeconds;
      _saveTimersToStorage();
      notifyListeners();
    }
  }

  @override
  void dispose() {
    _globalTicker?.cancel();
    super.dispose();
  }
}

class TimerDashboard extends StatelessWidget {
  const TimerDashboard({super.key});

  @override
  Widget build(BuildContext context) {
    final state = context.watch<TimerAppState>();

    return Scaffold(
      appBar: AppBar(
        title: const Text(
          "Heather's Timer", 
          style: TextStyle(fontWeight: FontWeight.w800, letterSpacing: 0.6, fontSize: 24)
        ),
        centerTitle: false,
        backgroundColor: Colors.transparent,
        elevation: 0,
      ),
      body: Padding(
        padding: const EdgeInsets.fromLTRB(16.0, 8.0, 16.0, 16.0),
        child: GridView.builder(
          itemCount: state.timers.length,
          gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: 2,
            crossAxisSpacing: 14.0,
            mainAxisSpacing: 14.0,
            childAspectRatio: 0.85,
          ),
          itemBuilder: (context, index) {
            return TimerCard(timer: state.timers[index]);
          },
        ),
      ),
    );
  }
}

class TimerCard extends StatelessWidget {
  final TimerModel timer;
  const TimerCard({super.key, required this.timer});

  @override
  Widget build(BuildContext context) {
    final appState = context.read<TimerAppState>();
    final theme = Theme.of(context);
    final bool isFinished = timer.remainingSeconds == 0;

    return Container(
      decoration: BoxDecoration(
        color: theme.cardColor,
        borderRadius: BorderRadius.circular(24),
        border: Border.all(
          color: timer.isRunning 
              ? theme.colorScheme.primary.withOpacity(0.4) 
              : Colors.white.withOpacity(0.05),
          width: 1.5,
        ),
      ),
      padding: const EdgeInsets.all(16.0),
      child: Column(
        mainAxisAlignment: MainAxisAlignment.spaceBetween,
        children: [
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Expanded(
                child: Text(
                  timer.label,
                  style: TextStyle(
                    color: Colors.white.withOpacity(0.6), 
                    fontSize: 14, 
                    fontWeight: FontWeight.w600
                  ),
                  overflow: TextOverflow.ellipsis,
                ),
              ),
              if (isFinished)
                const Icon(Icons.check_circle_rounded, color: Colors.greenAccent, size: 20),
            ],
          ),
          Stack(
            alignment: Alignment.center,
            children: [
              SizedBox(
                height: 95,
                width: 95,
                child: CircularProgressIndicator(
                  value: timer.progress,
                  strokeWidth: 5.5,
                  backgroundColor: Colors.white.withOpacity(0.04),
                  valueColor: AlwaysStoppedAnimation<Color>(
                    timer.isRunning 
                        ? theme.colorScheme.primary 
                        : theme.colorScheme.primary.withOpacity(0.25),
                  ),
                ),
              ),
              Text(
                timer.formattedTime,
                style: TextStyle(
                  fontSize: 21, 
                  fontWeight: FontWeight.bold, 
                  color: isFinished ? Colors.white.withOpacity(0.4) : Colors.white
                ),
              ),
            ],
          ),
          Row(
            children: [
              Expanded(
                child: ElevatedButton(
                  style: ElevatedButton.styleFrom(
                    backgroundColor: timer.isRunning 
                        ? Colors.amber.withOpacity(0.12) 
                        : theme.colorScheme.primary.withOpacity(0.12),
                    foregroundColor: timer.isRunning ? Colors.amber : theme.colorScheme.primary,
                    elevation: 0,
                    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                    padding: const EdgeInsets.symmetric(vertical: 10),
                  ),
                  onPressed: () => appState.toggleTimer(timer.id),
                  child: Icon(timer.isRunning ? Icons.pause_rounded : Icons.play_arrow_rounded, size: 22),
                ),
              ),
              const SizedBox(width: 8),
              IconButton(
                style: IconButton.styleFrom(
                  backgroundColor: Colors.white.withOpacity(0.03),
                  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
                  padding: const EdgeInsets.all(10),
                ),
                icon: Icon(Icons.refresh_rounded, size: 18, color: Colors.white.withOpacity(0.4)),
                onPressed: () => appState.resetTimer(timer.id),
              ),
            ],
          )
        ],
      ),
    );
  }
}
