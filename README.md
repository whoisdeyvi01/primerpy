import 'dart:async';
import 'package:flutter/material.dart';
import 'dart:math';

void main() {
  runApp(MaterialApp(
    home: SnakeGame(),
    debugShowCheckedModeBanner: false,
  ));
}

class SnakeGame extends StatefulWidget {
  @override
  _SnakeGameState createState() => _SnakeGameState();
}

class _SnakeGameState extends State<SnakeGame> {
  final int rowCount = 15;
  final int columnCount = 15;
  final int squareSize = 20;

  List<Point<int>> snake = [Point(7, 7)];
  Point<int> food = Point(5, 5);
  String direction = 'up';
  Timer? timer;

  @override
  void initState() {
    super.initState();
    startGame();
  }

  void startGame() {
    snake = [Point(7, 7)];
    createFood();
    direction = 'up';

    timer?.cancel();
    timer = Timer.periodic(Duration(milliseconds: 300), (_) {
      setState(() {
        moveSnake();
      });
    });
  }

  void createFood() {
    Random random = Random();
    food = Point(random.nextInt(columnCount), random.nextInt(rowCount));
    while (snake.contains(food)) {
      food = Point(random.nextInt(columnCount), random.nextInt(rowCount));
    }
  }

  void moveSnake() {
    Point<int> newHead;
    Point<int> currentHead = snake.first;

    switch (direction) {
      case 'up':
        newHead = Point(currentHead.x, (currentHead.y - 1 + rowCount) % rowCount);
        break;
      case 'down':
        newHead = Point(currentHead.x, (currentHead.y + 1) % rowCount);
        break;
      case 'left':
        newHead = Point((currentHead.x - 1 + columnCount) % columnCount, currentHead.y);
        break;
      case 'right':
        newHead = Point((currentHead.x + 1) % columnCount, currentHead.y);
        break;
      default:
        newHead = currentHead;
    }

    if (snake.contains(newHead)) {
      timer?.cancel();
      showGameOverDialog();
      return;
    }

    snake.insert(0, newHead);

    if (newHead == food) {
      createFood();
    } else {
      snake.removeLast();
    }
  }

  void showGameOverDialog() {
    showDialog(
      context: context,
      builder: (context) => AlertDialog(
        title: Text('Juego Terminado'),
        content: Text('¡Has chocado con el gusanito!'),
        actions: [
          TextButton(
            onPressed: () {
              Navigator.of(context).pop();
              startGame();
            },
            child: Text('Reiniciar'),
          ),
        ],
      ),
    );
  }

  void changeDirection(String newDirection) {
    if ((direction == 'up' && newDirection == 'down') ||
        (direction == 'down' && newDirection == 'up') ||
        (direction == 'left' && newDirection == 'right') ||
        (direction == 'right' && newDirection == 'left')) {
      return; // evitar ir en dirección opuesta directa
    }
    direction = newDirection;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('Juego del Gusanito')),
      body: Column(
        children: [
          Expanded(
            child: Container(
              color: Colors.black,
              child: GridView.builder(
                physics: NeverScrollableScrollPhysics(),
                gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                  crossAxisCount: columnCount,
                ),
                itemCount: rowCount * columnCount,
                itemBuilder: (context, index) {
                  int x = index % columnCount;
                  int y = index ~/ columnCount;
                  Point<int> point = Point(x, y);

                  Color color;
                  if (snake.first == point) {
                    color = Colors.greenAccent;
                  } else if (snake.contains(point)) {
                    color = Colors.green;
                  } else if (point == food) {
                    color = Colors.red;
                  } else {
                    color = Colors.grey[900]!;
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
          ),
          Container(
            height: 150,
            color: Colors.black12,
            child: Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                IconButton(
                  iconSize: 48,
                  icon: Icon(Icons.arrow_upward),
                  onPressed: () => changeDirection('up'),
                ),
                Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    IconButton(
                      iconSize: 48,
                      icon: Icon(Icons.arrow_back),
                      onPressed: () => changeDirection('left'),
                    ),
                    SizedBox(height: 65),
                    IconButton(
                      iconSize: 48,
                      icon: Icon(Icons.arrow_forward),
                      onPressed: () => changeDirection('right'),
                    ),
                  ],
                ),
                IconButton(
                  iconSize: 48,
                  icon: Icon(Icons.arrow_downward),
                  onPressed: () => changeDirection('down'),
                ),
              ],
            ),
          )
        ],
      ),
    );
  }
}
