 import 'dart:async';
import 'dart:math';
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Snake Game',
      home: SnakeGame(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class SnakeGame extends StatefulWidget {
  @override
  _SnakeGameState createState() => _SnakeGameState();
}

class _SnakeGameState extends State<SnakeGame> {
  static const int rowCount = 20;
  static const int columnCount = 20;
  final random = Random();

  List<Point<int>> snake = [Point(10, 10)];
  Point<int> food = Point(5, 5);
  String direction = 'up';
  Timer? timer;

  @override
  void initState() {
    super.initState();
    startGame();
  }

  void startGame() {
    timer = Timer.periodic(Duration(milliseconds: 200), (_) {
      setState(() {
        moveSnake();
        if (snake.first == food) {
          snake.add(snake.last);
          food = Point(random.nextInt(columnCount), random.nextInt(rowCount));
        }
        checkGameOver();
      });
    });
  }

  void moveSnake() {
    final head = snake.first;
    Point<int> newHead;

    switch (direction) {
      case 'up':
        newHead = Point(head.x, head.y - 1);
        break;
      case 'down':
        newHead = Point(head.x, head.y + 1);
        break;
      case 'left':
        newHead = Point(head.x - 1, head.y);
        break;
      case 'right':
        newHead = Point(head.x + 1, head.y);
        break;
      default:
        return;
    }

    snake.insert(0, newHead);
    snake.removeLast();
  }

  void checkGameOver() {
    final head = snake.first;
    if (head.x < 0 || head.y < 0 || head.x >= columnCount || head.y >= rowCount || snake.skip(1).contains(head)) {
      timer?.cancel();
      showDialog(
        context: context,
        builder: (_) => AlertDialog(
          title: Text('Game Over'),
          content: Text('You lost!'),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.of(context).pop();
                resetGame();
              },
              child: Text('Restart'),
            ),
          ],
        ),
      );
    }
  }

  void resetGame() {
    setState(() {
      snake = [Point(10, 10)];
      food = Point(5, 5);
      direction = 'up';
      startGame();
    });
  }

  void changeDirection(String newDirection) {
    if ((direction == 'up' && newDirection != 'down') ||
        (direction == 'down' && newDirection != 'up') ||
        (direction == 'left' && newDirection != 'right') ||
        (direction == 'right' && newDirection != 'left')) {
      direction = newDirection;
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: GestureDetector(
        onVerticalDragUpdate: (details) {
          if (details.primaryDelta! < 0) changeDirection('up');
          else if (details.primaryDelta! > 0) changeDirection('down');
        },
        onHorizontalDragUpdate: (details) {
          if (details.primaryDelta! < 0) changeDirection('left');
          else if (details.primaryDelta! > 0) changeDirection('right');
        },
        child: GridView.builder(
          itemCount: rowCount * columnCount,
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: columnCount,
          ),
          itemBuilder: (context, index) {
            final x = index % columnCount;
            final y = index ~/ columnCount;
            final point = Point(x, y);

            Color color;
            if (snake.contains(point)) {
              color = Colors.green;
            } else if (point == food) {
              color = Colors.red;
            } else {
              color = Colors.grey.shade900;
            }

            return Container(
              margin: EdgeInsets.all(1),
              decoration: BoxDecoration(
                color: color,
                borderRadius: BorderRadius.circular(3),
              ),
            );
          },
        ),
      ),
    );
  }
}
