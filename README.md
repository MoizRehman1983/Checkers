package com.example.demo12;

import javafx.application.Application;
import javafx.geometry.Insets;
import javafx.geometry.Pos;
import javafx.scene.Group;
import javafx.scene.Parent;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.layout.Background;
import javafx.scene.layout.BackgroundFill;
import javafx.scene.layout.CornerRadii;
import javafx.scene.layout.Pane;
import javafx.scene.layout.StackPane;
import javafx.scene.layout.VBox;
import javafx.scene.paint.Color;
import javafx.scene.paint.Paint;
import javafx.scene.shape.Ellipse;
import javafx.scene.shape.Rectangle;
import javafx.scene.text.Font;
import javafx.scene.text.Text;
import javafx.stage.Stage;

public class CheckersApp extends Application {

    // Constants for tile size and board dimensions
    public static final int TILE_SIZE = 100;
    public static final int WIDTH = 8;
    public static final int HEIGHT = 8;

    // Array representing the game board
    private Tile[][] board = new Tile[WIDTH][HEIGHT];
    private Group tileGroup = new Group();
    private Group pieceGroup = new Group();
    private Text statusText = new Text();
    private Button replayButton = new Button("Replay");
    private Button exitButton = new Button("Exit");

    // Current turn and piece for multi-capture sequence
    private PieceType currentTurn = PieceType.WHITE;
    private Piece currentPiece = null;

    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        // Show the main menu first
        primaryStage.setScene(createMainMenuScene(primaryStage));
        primaryStage.setTitle("checkers");
        primaryStage.show();
    }

    // Main menu scene
    private Scene createMainMenuScene(Stage primaryStage) {
        StackPane root = new StackPane();
        root.setBackground(new Background(new BackgroundFill(Color.DARKGREEN, CornerRadii.EMPTY, Insets.EMPTY)));

        VBox vbox = new VBox(30);
        vbox.setAlignment(Pos.CENTER);

        Text checkerLabel = new Text("CHECKERS");
        checkerLabel.setFont(Font.font("Impact", 150));
        checkerLabel.setFill(Color.RED);
        checkerLabel.setStroke(Color.BLACK);
        checkerLabel.setStrokeWidth(2);

        ImageView freeSpaceImageView = new ImageView(new Image("file:C:/Users/Hobopk/Desktop/piece.PNG"));
        freeSpaceImageView.setFitWidth(100);
        freeSpaceImageView.setFitHeight(100);

        Button playButton = createStyledButton("▶ Play");
        playButton.setOnAction(e -> primaryStage.setScene(createGameModeScene(primaryStage)));

        vbox.getChildren().addAll(checkerLabel, freeSpaceImageView, playButton);
        root.getChildren().add(vbox);

        return new Scene(root, 800, 600);
    }

    // Game mode scene
    private Scene createGameModeScene(Stage primaryStage) {
        StackPane root = new StackPane();
        root.setBackground(new Background(new BackgroundFill(Color.DARKGREEN, CornerRadii.EMPTY, Insets.EMPTY)));

        VBox vbox = new VBox(20);
        vbox.setAlignment(Pos.CENTER);

        Text gameModeText = new Text("Game Mode");
        gameModeText.setFont(new Font("Impact", 50));
        gameModeText.setFill(Color.RED);
        gameModeText.setStroke(Color.BLACK);
        gameModeText.setStrokeWidth(2);

        Button twoPlayerButton = createStyledButton("2 Player");
        twoPlayerButton.setFont(new Font("Arial", 36)); // Larger font size
        twoPlayerButton.setStyle("-fx-background-color: black; -fx-text-fill: white; -fx-padding: 20 40 20 40;"); // Larger padding
        twoPlayerButton.setOnAction(e -> primaryStage.setScene(new Scene(createGameContent())));

        Text creditsText = new Text("\n\n\nCREATED BY: MUHAMMAD ALI, MOIZ REHMAN, HARIS ALI");
        creditsText.setFont(new Font("Aptos Display", 20));
        creditsText.setFill(Color.WHITE);

        vbox.getChildren().addAll(gameModeText, twoPlayerButton, creditsText);
        root.getChildren().add(vbox);

        return new Scene(root, 800, 600);
    }

    // Method to create the game content
    private Parent createGameContent() {
        Pane root = new Pane();
        root.setPrefSize(WIDTH * TILE_SIZE, HEIGHT * TILE_SIZE +80 );
        root.getChildren().addAll(tileGroup, pieceGroup, statusText, replayButton, exitButton);

        // Status text setup
        statusText.setFont(Font.font(36));
        statusText.setFill(Paint.valueOf("#000"));
        statusText.setTranslateX(10);
        statusText.setTranslateY(HEIGHT * TILE_SIZE + 40);
        updateStatusText();

        // Replay button setup
        replayButton.setTranslateX(WIDTH * TILE_SIZE - 200);
        replayButton.setTranslateY(HEIGHT * TILE_SIZE + 30);
        replayButton.setVisible(false);
        replayButton.setOnAction(e -> resetGame());

        // Exit button setup
        exitButton.setTranslateX(WIDTH * TILE_SIZE - 100);
        exitButton.setTranslateY(HEIGHT * TILE_SIZE + 30);
        exitButton.setVisible(false);
        exitButton.setOnAction(e -> exitGame());

        // Initialize the game board
        initializeBoard();

        return root;
    }

    // Method to initialize the game board
    private void initializeBoard() {
        tileGroup.getChildren().clear();
        pieceGroup.getChildren().clear();
        //setting color(alternative) for tiles
        for (int y = 0; y < HEIGHT; y++) {
            for (int x = 0; x < WIDTH; x++) {
                Tile tile = new Tile((x + y) % 2 == 0, x, y);
                board[x][y] = tile;
                tileGroup.getChildren().add(tile);
            //placing pieces(red & white)
                Piece piece = null;
                if (y <= 2 && (x + y) % 2 != 0) {
                    piece = makePiece(PieceType.RED, x, y);
                } else if (y >= 5 && (x + y) % 2 != 0) {
                    piece = makePiece(PieceType.WHITE, x, y);
                }

                if (piece != null) {
                    tile.setPiece(piece);
                    pieceGroup.getChildren().add(piece);
                }
            }
        }
    }
    // Method to attempt a move
    private MoveResult tryMove(Piece piece, int newX, int newY) {
        //condition for invalid move
        if (board[newX][newY].hasPiece() || (newX + newY) % 2 == 0) {
            return new MoveResult(MoveType.NONE);
        }
    //conversion for pixel to board coordinates
        int x0 = toBoard(piece.getOldX());
        int y0 = toBoard(piece.getOldY());
    //condition for king piece
        if (piece.isKinged()) {
            //condition to play normal kinged piece move
            if (Math.abs(newX - x0) == 1 && Math.abs(newY - y0) == 1) {
                return new MoveResult(MoveType.NORMAL);
            }
            //condition to capture using kinged piece
            else if (Math.abs(newX - x0) == 2 && Math.abs(newY - y0) == 2) {
                int x1 = x0 + (newX - x0) / 2;
                int y1 = y0 + (newY - y0) / 2;

                if (board[x1][y1].hasPiece() && board[x1][y1].getPiece().getType() != piece.getType()) {
                    return new MoveResult(MoveType.KILL, board[x1][y1].getPiece());
                }
            }
        } else {
            //move for  normal piece
            int moveDir = piece.getType().moveDir;
            if (Math.abs(newX - x0) == 1 && (newY - y0) == moveDir) {
                return new MoveResult(MoveType.NORMAL);
            }
            //condition to capture using normal piece
            else if (Math.abs(newX - x0) == 2 && (newY - y0) == moveDir * 2) {
                int x1 = x0 + (newX - x0) / 2;
                int y1 = y0 + (newY - y0) / 2;

                if (board[x1][y1].hasPiece() && board[x1][y1].getPiece().getType() != piece.getType()) {
                    return new MoveResult(MoveType.KILL, board[x1][y1].getPiece());
                }
            }
        }

        return new MoveResult(MoveType.NONE);
    }
    // Method to convert pixel coordinates to board coordinates
    private int toBoard(double pixel) {
        return (int) (pixel + TILE_SIZE / 2) / TILE_SIZE;
    }
    // Method to create a piece
    private Piece makePiece(PieceType type, int x, int y) {
        Piece piece = new Piece(type, x, y);

        piece.setOnMouseReleased(e -> {
            if (piece.getType() != currentTurn && currentPiece == null) {
                piece.abortMove();
                return;
            }

            int newX = toBoard(piece.getLayoutX());
            int newY = toBoard(piece.getLayoutY());

            MoveResult result = (newX < 0 || newY < 0 || newX >= WIDTH || newY >= HEIGHT)
                    ? new MoveResult(MoveType.NONE)
                    : tryMove(piece, newX, newY);

            int x0 = toBoard(piece.getOldX());
            int y0 = toBoard(piece.getOldY());

            switch (result.getType()) {
                case NONE:
                    piece.abortMove();
                    break;
                case NORMAL:
                    //checkes for possible capture moves
                    if (hasPossibleCaptureMoves(currentTurn)) {
                        statusText.setText("Invalid move! You must capture.");
                        //reset piece to it's original position
                        piece.abortMove();
                    }
                    //plays move
                    else {
                        piece.move(newX, newY);
                        board[x0][y0].setPiece(null);
                        board[newX][newY].setPiece(piece);
                        checkKing(piece, newY);
                        if (currentPiece == null) nextTurn();
                        currentPiece = null;
                    }
                    break;
                case KILL:
                    piece.move(newX, newY);
                    board[x0][y0].setPiece(null);
                    board[newX][newY].setPiece(piece);

                    Piece otherPiece = result.getPiece();
                    board[toBoard(otherPiece.getOldX())][toBoard(otherPiece.getOldY())].setPiece(null);
                    pieceGroup.getChildren().remove(otherPiece);
                    checkKing(piece, newY);
                    if (hasKillMove(piece)) {
                        currentPiece = piece;
                    } else {
                        nextTurn();
                        currentPiece = null;
                    }
                    break;
            }

            checkWinCondition();
        }
        );
        return piece;
    }
    // Method to check if a piece has a kill move
    private boolean hasKillMove(Piece piece) {
        int x = toBoard(piece.getOldX());
        int y = toBoard(piece.getOldY());

        int[][] directions = {
                {2, 2}, {2, -2}, {-2, 2}, {-2, -2}
        };

        for (int[] dir : directions) {
            int newX = x + dir[0];
            int newY = y + dir[1];
            if (newX >= 0 && newY >= 0 && newX < WIDTH && newY < HEIGHT) {
                if (tryMove(piece, newX, newY).getType() == MoveType.KILL) {
                    return true;
                }
            }
        }

        return false;
    }
    // Method to check if there are any possible capture moves for the current player
    private boolean hasPossibleCaptureMoves(PieceType player) {
        for (int y = 0; y < HEIGHT; y++) {
            for (int x = 0; x < WIDTH; x++) {
                Piece piece = board[x][y].getPiece();
                if (piece != null && piece.getType() == player) {
                    if (hasKillMove(piece)) {
                        return true;
                    }
                }
            }
        }
        return false;
    }

    // Method to check if a piece becomes a king
    private void checkKing(Piece piece, int newY) {
        if ((piece.getType() == PieceType.RED && newY == HEIGHT - 1) ||
                (piece.getType() == PieceType.WHITE && newY == 0)) {
            piece.setKinged(true);
        }
    }
    // Method to switch to the next turn
    private void nextTurn() {
        //currentTurn is RED then WHITE will be assigned to currentTurn for next move and text will be updated
        currentTurn = (currentTurn == PieceType.WHITE) ? PieceType.RED : PieceType.WHITE;
        updateStatusText();
    }
    // Method to update the status text based on the current turn
    private void updateStatusText() {
        statusText.setText("Current Turn: " + (currentTurn == PieceType.RED ? "Player 1 (Red)" : "Player 2 (White)"));
    }
    // Method to check the win condition
    private void checkWinCondition() {
        boolean redHasPieces = false;
        boolean whiteHasPieces = false;

        for (int y = 0; y < HEIGHT; y++) {
            for (int x = 0; x < WIDTH; x++) {
                Piece piece = board[x][y].getPiece();
                if (piece != null) {
                    if (piece.getType() == PieceType.RED) {
                        redHasPieces = true;
                    } else if (piece.getType() == PieceType.WHITE) {
                        whiteHasPieces = true;
                    }
                }
            }
        }

        if (!redHasPieces) {
            statusText.setText("Player 2 (White) Wins!");
            replayButton.setVisible(true);
            exitButton.setVisible(true);
        } else if (!whiteHasPieces) {
            statusText.setText("Player 1 (Red) Wins!");
            replayButton.setVisible(true);
            exitButton.setVisible(true);
        }
    }

    // Method to reset the game
    private void resetGame() {
        currentTurn = PieceType.WHITE;
        currentPiece = null;
        updateStatusText();
        replayButton.setVisible(false);
        exitButton.setVisible(false);
        initializeBoard();
    }

    // Method to exit the game
    private void exitGame() {
        Stage stage = (Stage) exitButton.getScene().getWindow();
        stage.close();
    }

    // Utility method to create a styled button
    private Button createStyledButton(String text) {
        Button button = new Button(text);
        button.setStyle("-fx-background-color: black; -fx-text-fill: white;");
        button.setFont(new Font("Arial", 24));
        return button;
    }
}
// Class representing the result of a move
class MoveResult {

    private MoveType type;
    private Piece piece;

    public MoveType getType() {
        return type;
    }

    public Piece getPiece() {
        return piece;
    }

    public MoveResult(MoveType type) {
        this(type, null);
    }

    public MoveResult(MoveType type, Piece piece) {
        this.type = type;
        this.piece = piece;
    }
}

// Enumeration representing types of moves
enum MoveType {
    NONE, NORMAL, KILL
}
// Class representing a piece
class Piece extends StackPane {

    private PieceType type;
    private double mouseX, mouseY;
    private double oldX, oldY;
    private boolean kinged = false;

    // Getters and setters
    public PieceType getType() {
        return type;
    }

    public double getOldX() {
        return oldX;
    }

    public double getOldY() {
        return oldY;
    }

    public boolean isKinged() {
        return kinged;
    }

    public void setKinged(boolean kinged) {
        this.kinged = kinged;
        Ellipse crown = new Ellipse(CheckersApp.TILE_SIZE * 0.15, CheckersApp.TILE_SIZE * 0.15);
        crown.setFill(Color.GOLD);
        crown.setStroke(Color.BLACK);
        crown.setStrokeWidth(CheckersApp.TILE_SIZE * 0.03);
        crown.setTranslateX(CheckersApp.TILE_SIZE * 0.5);
        crown.setTranslateY(CheckersApp.TILE_SIZE * 0.5);
        getChildren().add(crown);
    }
    // Constructor
    public Piece(PieceType type, int x, int y) {
        this.type = type;
        move(x, y);

        // Setup piece appearance
        Ellipse bg = new Ellipse(CheckersApp.TILE_SIZE * 0.3125, CheckersApp.TILE_SIZE * 0.26);
        bg.setFill(Color.BLACK);

        bg.setStroke(Color.BLACK);
        bg.setStrokeWidth(CheckersApp.TILE_SIZE * 0.03);

        bg.setTranslateX((CheckersApp.TILE_SIZE - CheckersApp.TILE_SIZE * 0.3125 * 2) / 2);
        bg.setTranslateY((CheckersApp.TILE_SIZE - CheckersApp.TILE_SIZE * 0.26 * 2) / 2 + CheckersApp.TILE_SIZE * 0.07);

        Ellipse ellipse = new Ellipse(CheckersApp.TILE_SIZE * 0.3125, CheckersApp.TILE_SIZE * 0.26);
        ellipse.setFill(type == PieceType.RED
                ? Color.valueOf("#c40003") : Color.valueOf("#fff9f4"));

        ellipse.setStroke(Color.BLACK);
        ellipse.setStrokeWidth(CheckersApp.TILE_SIZE * 0.03);

        ellipse.setTranslateX((CheckersApp.TILE_SIZE - CheckersApp.TILE_SIZE * 0.3125 * 2) / 2);
        ellipse.setTranslateY((CheckersApp.TILE_SIZE - CheckersApp.TILE_SIZE * 0.26 * 2) / 2);

        getChildren().addAll(bg, ellipse);

        // Mouse event handlers
        setOnMousePressed(e -> {
            mouseX = e.getSceneX();
            mouseY = e.getSceneY();
        });

        setOnMouseDragged(e -> {
            relocate(e.getSceneX() - mouseX + oldX, e.getSceneY() - mouseY + oldY);
        });
    }

    // Method to move the piece
    public void move(int x, int y) {
        oldX = x * CheckersApp.TILE_SIZE;
        oldY = y * CheckersApp.TILE_SIZE;
        relocate(oldX, oldY);
    }

    // Method to abort the move of the piece
    public void abortMove() {
        relocate(oldX, oldY);
    }
}

// Enumeration representing types of pieces
enum PieceType {
    RED(1), WHITE(-1);

    final int moveDir;

    PieceType(int moveDir) {
        this.moveDir = moveDir;
    }
}

// Class representing a tile on the board
class Tile extends StackPane {

    private Piece piece;

    public boolean hasPiece() {
        return piece != null;
    }

    public Piece getPiece() {
        return piece;
    }

    public void setPiece(Piece piece) {
        this.piece = piece;
    }

    public Tile(boolean light, int x, int y) {
        setWidth(CheckersApp.TILE_SIZE);
        setHeight(CheckersApp.TILE_SIZE);

        relocate(x * CheckersApp.TILE_SIZE, y * CheckersApp.TILE_SIZE);

        Rectangle border = new Rectangle(CheckersApp.TILE_SIZE, CheckersApp.TILE_SIZE);
        border.setFill(light ? Color.valueOf("#feb") : Color.valueOf("#582"));
        getChildren().add(border);
    }
}
