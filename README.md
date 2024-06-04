# SFML-Tennis 기능 추가
---------
제가 구현한 기능은 4번 주고받으면 바의 크기가 줄어드는 것과 2번 주고받을 때마다 공의 속도가 증가하는 것입니다.

1. 바가 줄어드는 장면
   
![크기 작아지는거](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/571b05d5-8879-495b-bff6-83aeaf3e9c9c)

바의 크기를 조절하는 함수입니다. 줄어들 수 있는 최대치를 만들어 일정치 이상 작아지면 더 이상 작아질 수 없게 만들었습니다.

![크기 조절함수](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/45a92c74-8410-4868-857b-99a0954be0df)

2. 공이 빨라지는 장면

![123](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/aea4caa7-1140-404a-945d-d6a5fd0fc327)


두 번 정도 튕길 때마다 공이 점점 빨라지도록 설계하였습니다.

![스피드 조절 함수](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/2ea5a9f6-3dd1-4f6d-8e0a-e44b08109434)

전체 구현 장면입니다.

![전체](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/1d3e587f-4081-47e2-83ab-2138f087617f)

득점이 생기면 다시 원래의 초기값을 가지도록 설계하였습니다.


## 코드
            ////////////////////////////////////////////////////////////
            // Headers
            ////////////////////////////////////////////////////////////
            #include <SFML/Graphics.hpp>
            #include <SFML/Audio.hpp>
            #include <cmath>
            #include <ctime>
            #include <cstdlib>
            
            #ifdef SFML_SYSTEM_IOS
            #include <SFML/Main.hpp>
            #endif
            
            std::string resourcesDir()
            {
            #ifdef SFML_SYSTEM_IOS
                return "";
            #else
                return "resources/";
            #endif
            }
            
            ////////////////////////////////////////////////////////////
            /// Entry point of application
            ///
            /// \return Application exit code
            ///
            ////////////////////////////////////////////////////////////
            int main()
            {
                std::srand(static_cast<unsigned int>(std::time(NULL)));
            
                // Define some constants
                const float pi = 3.14159f;
                const float gameWidth = 800;
                const float gameHeight = 600;
                sf::Vector2f paddleSize(25, 100);
                sf::Vector2f originalPaddleSize = paddleSize;
                float ballRadius = 10.f;
                int bounceCount = 0;
                int speedCount = 0;
                int max = 0;
                // Create the window of the application
                sf::RenderWindow window(sf::VideoMode(static_cast<unsigned int>(gameWidth), static_cast<unsigned int>(gameHeight), 32), "SFML Tennis",
                    sf::Style::Titlebar | sf::Style::Close);
                window.setVerticalSyncEnabled(true);
            
                // Load the sounds used in the game
                sf::SoundBuffer ballSoundBuffer;
                if (!ballSoundBuffer.loadFromFile(resourcesDir() + "ball.wav"))
                    return EXIT_FAILURE;
                sf::Sound ballSound(ballSoundBuffer);
            
                // Create the SFML logo texture:
                sf::Texture sfmlLogoTexture;
                if (!sfmlLogoTexture.loadFromFile(resourcesDir() + "sfml_logo.png"))
                    return EXIT_FAILURE;
                sf::Sprite sfmlLogo;
                sfmlLogo.setTexture(sfmlLogoTexture);
                sfmlLogo.setPosition(170, 50);
            
                // Create the left paddle
                sf::RectangleShape leftPaddle;
                leftPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
                leftPaddle.setOutlineThickness(3);
                leftPaddle.setOutlineColor(sf::Color::Black);
                leftPaddle.setFillColor(sf::Color(100, 100, 200));
                leftPaddle.setOrigin(paddleSize / 2.f);
            
                // Create the right paddle
                sf::RectangleShape rightPaddle;
                rightPaddle.setSize(paddleSize - sf::Vector2f(3, 3));
                rightPaddle.setOutlineThickness(3);
                rightPaddle.setOutlineColor(sf::Color::Black);
                rightPaddle.setFillColor(sf::Color(200, 100, 100));
                rightPaddle.setOrigin(paddleSize / 2.f);
            
                // Create the ball
                sf::CircleShape ball;
                ball.setRadius(ballRadius - 3);
                ball.setOutlineThickness(2);
                ball.setOutlineColor(sf::Color::Black);
                ball.setFillColor(sf::Color::White);
                ball.setOrigin(ballRadius / 2, ballRadius / 2);
            
                // Load the text font
                sf::Font font;
                if (!font.loadFromFile(resourcesDir() + "tuffy.ttf"))
                    return EXIT_FAILURE;
            
                // Initialize the pause message
                sf::Text pauseMessage;
                pauseMessage.setFont(font);
                pauseMessage.setCharacterSize(40);
                pauseMessage.setPosition(170.f, 200.f);
                pauseMessage.setFillColor(sf::Color::White);
            
            #ifdef SFML_SYSTEM_IOS
                pauseMessage.setString("Welcome to SFML Tennis!\nTouch the screen to start the game.");
            #else
                pauseMessage.setString("Welcome to SFML Tennis!\n\nPress space to start the game.");
            #endif
            
                // Initialize the score display
                sf::Text scoreMessage;
                scoreMessage.setFont(font);
                scoreMessage.setCharacterSize(30);
                scoreMessage.setPosition(gameWidth / 2.f - 50, 20.f);
                scoreMessage.setFillColor(sf::Color::White);
            
                int leftScore = 0;
                int rightScore = 0;
            
                // Define the paddles properties
                sf::Clock AITimer;
                const sf::Time AITime = sf::seconds(0.1f);
                const float paddleSpeed = 400.f;
                float rightPaddleSpeed = 0.f;
                float ballSpeed = 400.f;
                float ballAngle = 0.f; // to be changed later
            
                sf::Clock clock;
                bool isPlaying = false;
                while (window.isOpen())
                {
                    // Handle events
                    sf::Event event;
                    while (window.pollEvent(event))
                    {
                        // Window closed or escape key pressed: exit
                        if ((event.type == sf::Event::Closed) ||
                            ((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Escape)))
                        {
                            window.close();
                            break;
                        }
                        if (speedCount >= 2)
                        {
                            ballSpeed += 200; // 공의 속도 증가
                            speedCount = 0; // 스피드 카운트 초기화
                        }
                        // Space key pressed: play
                        if (((event.type == sf::Event::KeyPressed) && (event.key.code == sf::Keyboard::Space)) ||
                            (event.type == sf::Event::TouchBegan))
                        {
                            if (!isPlaying)
                            {
                                // (re)start the game
                                isPlaying = true;
                                clock.restart();
            
                                // Reset the position of the paddles and ball
                                leftPaddle.setPosition(10.f + paddleSize.x / 2.f, gameHeight / 2.f);
                                rightPaddle.setPosition(gameWidth - 10.f - paddleSize.x / 2.f, gameHeight / 2.f);
                                ball.setPosition(gameWidth / 2.f, gameHeight / 2.f);
            
                                // Reset the ball angle
                                do
                                {
                                    // Make sure the ball initial angle is not too much vertical
                                    ballAngle = static_cast<float>(std::rand() % 360) * 2.f * pi / 360.f;
                                } while (std::abs(std::cos(ballAngle)) < 0.7f);
            
                                // Reset the scores if game was won/lost
                                if (leftScore >= 5 || rightScore >= 5)
                                {
                                    leftScore = 0;
                                    rightScore = 0;
                                }
            
                                // Reset paddle sizes
                                leftPaddle.setSize(originalPaddleSize - sf::Vector2f(3, 3));
                                rightPaddle.setSize(originalPaddleSize - sf::Vector2f(3, 3));
                                leftPaddle.setOrigin(leftPaddle.getSize() / 2.f);
                                rightPaddle.setOrigin(rightPaddle.getSize() / 2.f);
                            }
                        }
            
                        // Window size changed
                        // Window size changed, adjust view appropriately
                        if (event.type == sf::Event::Resized)
                        {
                            sf::FloatRect visibleArea(0, 0, event.size.width, event.size.height);
                            window.setView(sf::View(visibleArea));
                        }
                    }
            
                    if (isPlaying)
                    {
                        float deltaTime = clock.restart().asSeconds();
            
                        // Move the player's paddle
                        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) &&
                            (leftPaddle.getPosition().y - paddleSize.y / 2 > 5.f))
                        {
                            leftPaddle.move(0.f, -paddleSpeed * deltaTime);
                        }
                        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Down) &&
                            (leftPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f))
                        {
                            leftPaddle.move(0.f, paddleSpeed * deltaTime);
                        }
            
                        if (sf::Touch::isDown(0))
                        {
                            sf::Vector2i pos = sf::Touch::getPosition(0);
                            sf::Vector2f mappedPos = window.mapPixelToCoords(pos);
                            leftPaddle.setPosition(leftPaddle.getPosition().x, mappedPos.y);
                        }
            
                        // Move the computer's paddle
                        if (((rightPaddleSpeed < 0.f) && (rightPaddle.getPosition().y - paddleSize.y / 2 > 5.f)) ||
                            ((rightPaddleSpeed > 0.f) && (rightPaddle.getPosition().y + paddleSize.y / 2 < gameHeight - 5.f)))
                        {
                            rightPaddle.move(0.f, rightPaddleSpeed * deltaTime);
                        }
            
                        // Update the computer's paddle direction according to the ball position
                        if (AITimer.getElapsedTime() > AITime)
                        {
                            AITimer.restart();
                            if (ball.getPosition().y + ballRadius > rightPaddle.getPosition().y + paddleSize.y / 2)
                                rightPaddleSpeed = paddleSpeed;
                            else if (ball.getPosition().y - ballRadius < rightPaddle.getPosition().y - paddleSize.y / 2)
                                rightPaddleSpeed = -paddleSpeed;
                            else
                                rightPaddleSpeed = 0.f;
                        }
            
                        // Move the ball
                        float factor = ballSpeed * deltaTime;
                        ball.move(std::cos(ballAngle) * factor, std::sin(ballAngle) * factor);
            
            #ifdef SFML_SYSTEM_IOS
                        const std::string inputString = "Touch the screen to restart.";
            #else
                        const std::string inputString = "Press space to restart or\nescape to exit.";
            #endif
            
                        // Check collisions between the ball and the screen
                        if (ball.getPosition().x - ballRadius < 0.f)
                        {
                            rightScore++;
                            isPlaying = false;
                            if (rightScore >= 5)
                            {
                                pauseMessage.setString("You Lost the Match!\n\n" + inputString);
                                leftScore = 0;
                                rightScore = 0;
                            }
                            else
                            {
                                pauseMessage.setString("You Lost the Point!\n\n" + inputString);
                            }
                            // Reset paddle sizes
                            leftPaddle.setSize(originalPaddleSize - sf::Vector2f(3, 3));
                            rightPaddle.setSize(originalPaddleSize - sf::Vector2f(3, 3));
                            bounceCount = 0;
                        }
                        if (ball.getPosition().x + ballRadius > gameWidth)
                        {
                            leftScore++;
                            isPlaying = false;
                            if (leftScore >= 5)
                            {
                                pauseMessage.setString("You Won the Match!\n\n" + inputString);
                                leftScore = 0;
                                rightScore = 0;
                            }
                            else
                            {
                                pauseMessage.setString("You Won the Point!\n\n" + inputString);
                            }
                            // Reset paddle sizes
                            leftPaddle.setSize(originalPaddleSize - sf::Vector2f(3, 3));
                            rightPaddle.setSize(originalPaddleSize - sf::Vector2f(3, 3));
                            bounceCount = 0;
                            ballSpeed = 400;
                        }
                        if (ball.getPosition().y - ballRadius < 0.f)
                        {
                            ballSound.play();
                            ballAngle = -ballAngle;
                            ball.setPosition(ball.getPosition().x, ballRadius + 0.1f);
                        }
                        if (ball.getPosition().y + ballRadius > gameHeight)
                        {
                            ballSound.play();
                            ballAngle = -ballAngle;
                            ball.setPosition(ball.getPosition().x, gameHeight - ballRadius - 0.1f);
                        }
            
                        // Check the collisions between the ball and the paddles
                        // Left Paddle
                        if (ball.getPosition().x - ballRadius < leftPaddle.getPosition().x + leftPaddle.getSize().x / 2 &&
                            ball.getPosition().x - ballRadius > leftPaddle.getPosition().x - leftPaddle.getSize().x / 2 &&
                            ball.getPosition().y + ballRadius >= leftPaddle.getPosition().y - leftPaddle.getSize().y / 2 &&
                            ball.getPosition().y - ballRadius <= leftPaddle.getPosition().y + leftPaddle.getSize().y / 2)
                        {
                            if (ball.getPosition().y > leftPaddle.getPosition().y)
                                ballAngle = pi - ballAngle + static_cast<float>(std::rand() % 20) * pi / 180;
                            else
                                ballAngle = pi - ballAngle - static_cast<float>(std::rand() % 20) * pi / 180;
            
                            ballSound.play();
                            ball.setPosition(leftPaddle.getPosition().x + ballRadius + leftPaddle.getSize().x / 2 + 0.1f, ball.getPosition().y);
                            bounceCount++;
                            speedCount++;
                        }
            
                        // Right Paddle
                        if (ball.getPosition().x + ballRadius > rightPaddle.getPosition().x - rightPaddle.getSize().x / 2 &&
                            ball.getPosition().x + ballRadius < rightPaddle.getPosition().x + rightPaddle.getSize().x / 2 &&
                            ball.getPosition().y + ballRadius >= rightPaddle.getPosition().y - rightPaddle.getSize().y / 2 &&
                            ball.getPosition().y - ballRadius <= rightPaddle.getPosition().y + rightPaddle.getSize().y / 2)
                        {
                            if (ball.getPosition().y > rightPaddle.getPosition().y)
                                ballAngle = pi - ballAngle + static_cast<float>(std::rand() % 20) * pi / 180;
                            else
                                ballAngle = pi - ballAngle - static_cast<float>(std::rand() % 20) * pi / 180;
            
                            ballSound.play();
                            ball.setPosition(rightPaddle.getPosition().x - ballRadius - rightPaddle.getSize().x / 2 - 0.1f, ball.getPosition().y);
                            bounceCount++;
                        }
            
                        // Shrink paddles if bounce count is 4
                        if (bounceCount >= 4 && max<3)
                        {
                            bounceCount = 0;
                            leftPaddle.setSize(leftPaddle.getSize() - sf::Vector2f(0, 30));
                            rightPaddle.setSize(rightPaddle.getSize() - sf::Vector2f(0, 30));
                            leftPaddle.setOrigin(leftPaddle.getSize() / 2.f);
                            rightPaddle.setOrigin(rightPaddle.getSize() / 2.f);
                            max++;
                        }
                    }
            
                    // Clear the window
                    window.clear(sf::Color(50, 50, 50));
            
                    if (isPlaying)
                    {
                        // Draw the paddles and the ball
                        window.draw(leftPaddle);
                        window.draw(rightPaddle);
                        window.draw(ball);
                    }
                    else
                    {
                        // Draw the pause message
                        window.draw(pauseMessage);
                        window.draw(sfmlLogo);
                    }
            
                    // Display the score
                    scoreMessage.setString(std::to_string(leftScore) + " : " + std::to_string(rightScore));
                    window.draw(scoreMessage);
            
                    // Display things on screen
                    window.display();
                }
            
                return EXIT_SUCCESS;
            }



