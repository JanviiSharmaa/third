#include <SFML/Graphics.hpp>
#include <vector>
#include <cstdlib>
#include <ctime>

using namespace sf;
using namespace std;

const int WIDTH = 800;
const int HEIGHT = 600;
const int SIZE = 20;

enum Direction { UP, DOWN, LEFT, RIGHT };

struct Segment {
    int x, y;
};

class SnakeGame {
public:
    SnakeGame()
        : window(VideoMode(WIDTH, HEIGHT), "Snake Game"), direction(RIGHT), gameOver(false) {
        window.setFramerateLimit(10);
        snake.push_back({ WIDTH / 2, HEIGHT / 2 });
        generateFood();
    }

    void run() {
        while (window.isOpen()) {
            handleEvents();
            if (!gameOver) {
                update();
            }
            render();
        }
    }

private:
    RenderWindow window;
    vector<Segment> snake;
    Segment food;
    Direction direction;
    bool gameOver;
    int score = 0;

    void handleEvents() {
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed)
                window.close();

            if (event.type == Event::KeyPressed) {
                if (event.key.code == Keyboard::Up && direction != DOWN) direction = UP;
                else if (event.key.code == Keyboard::Down && direction != UP) direction = DOWN;
                else if (event.key.code == Keyboard::Left && direction != RIGHT) direction = LEFT;
                else if (event.key.code == Keyboard::Right && direction != LEFT) direction = RIGHT;
            }
        }
    }

    void update() {
        Segment head = snake.front();
        switch (direction) {
            case UP: head.y -= SIZE; break;
            case DOWN: head.y += SIZE; break;
            case LEFT: head.x -= SIZE; break;
            case RIGHT: head.x += SIZE; break;
        }

        // Check collision with walls
        if (head.x < 0 || head.x >= WIDTH || head.y < 0 || head.y >= HEIGHT) {
            gameOver = true;
            return;
        }

        // Check self-collision
        for (const auto& segment : snake) {
            if (head.x == segment.x && head.y == segment.y) {
                gameOver = true;
                return;
            }
        }

        // Move snake
        snake.insert(snake.begin(), head);
        if (head.x == food.x && head.y == food.y) {
            score++;
            generateFood();
        } else {
            snake.pop_back();
        }
    }

    void render() {
        window.clear(Color::Black);

        // Draw snake
        for (const auto& segment : snake) {
            RectangleShape rect(Vector2f(SIZE - 1, SIZE - 1));
            rect.setPosition(segment.x, segment.y);
            rect.setFillColor(Color::Green);
            window.draw(rect);
        }

        // Draw food
        RectangleShape foodShape(Vector2f(SIZE - 1, SIZE - 1));
        foodShape.setPosition(food.x, food.y);
        foodShape.setFillColor(Color::Red);
        window.draw(foodShape);

        // Game Over
        if (gameOver) {
            Font font;
            if (font.loadFromFile("arial.ttf")) {
                Text gameOverText("GAME OVER", font, 40);
                gameOverText.setPosition(WIDTH / 2 - 120, HEIGHT / 2 - 20);
                gameOverText.setFillColor(Color::White);
                window.draw(gameOverText);
            }
        }

        window.display();
    }

    void generateFood() {
        srand(static_cast<unsigned>(time(0)));
        food.x = (rand() % (WIDTH / SIZE)) * SIZE;
        food.y = (rand() % (HEIGHT / SIZE)) * SIZE;
    }
};

int main() {
    SnakeGame game;
    game.run();
    return 0;
}
