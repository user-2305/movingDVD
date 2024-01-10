# movingDVD

Для реализации движения логотипа DVD по вашей панели на JavaFX, вам нужно будет написать следующий код:
```java
import javafx.animation.KeyFrame;
import javafx.animation.Timeline;
import javafx.application.Application;
import javafx.geometry.Point2D;
import javafx.scene.Scene;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.layout.Pane;
import javafx.scene.paint.Color;
import javafx.scene.shape.Rectangle;
import javafx.util.Duration;
import org.apache.commons.math3.analysis.interpolation.TwoPointInterpolator;
import org.apache.commons.math3.analysis.polynomials.PolynomialSplineFunction;
import org.apache.commons.math3.linear.RealMatrix;
import org.apache.commons.math3.linear.Array2DRowRealMatrix;

import java.io.File;

public class MovingLogo extends Application {

    private final double PANEL_WIDTH = 400;
    private final double PANEL_HEIGHT = 400;
    private final double IMAGE_WIDTH = 100;
    private final double IMAGE_HEIGHT = 100;
    private final String IMAGE_PATH = "logo.png";

    @Override
    public void start(java.awt.event.EventObject event) {
        Pane root = new Pane();
        root.setStyle("-fx-background-color: #FFFFFF;");

        Image image = new Image(new File(IMAGE_PATH).toURI().toString());
        ImageView imageView = new ImageView(image);
        imageView.setFitHeight(IMAGE_HEIGHT);
        imageView.setFitWidth(IMAGE_WIDTH);

        Rectangle panel = new Rectangle(PANEL_WIDTH, PANEL_HEIGHT, Color.TRANSPARENT);
        root.getChildren().addAll(panel, imageView);

        Timeline timeline = new Timeline(new KeyFrame(Duration.seconds(1), e -> {
            Point2D currentPoint = imageView.getLayoutBounds().getMinX() < panel.getWidth() - imageView.getWidth() ?
                    new Point2D.Double(imageView.getLayoutX(), imageView.getLayoutY()) :
                    new Point2D.Double(imageView.getLayoutX(), imageView.getLayoutY() + 5);
            Point2D targetPoint = new Point2D.Double(panel.getWidth() - imageView.getWidth(), panel.getHeight() - imageView.getHeight());

            RealMatrix coefficientsMatrix = new Array2DRowRealMatrix(new double[][] {
                    {currentPoint.getX(), 1},
                    {targetPoint.getX(), 1}
            });

            PolynomialSplineFunction splineFunction = new PolynomialSplineFunction(coefficientsMatrix, TwoPointInterpolator.interpolate(currentPoint, targetPoint));
            double newX = splineFunction.value(imageView.getLayoutX());
            imageView.setLayoutX(newX);

            if (imageView.getLayoutX() > panel.getWidth() - imageView.getWidth()) {
                imageView.setLayoutY(0);
                imageView.setLayoutX(0);
                imageView.setLayoutY(imageView.getLayoutY() + 5);
                if (imageView.getLayoutY() > panel.getHeight() - imageView.getHeight()) {
                    imageView.setLayoutY(0);
                }
            }
        }));
        timeline.setCycleCount(Timeline.INDEFINITE);
        timeline.play();

        Scene scene = new Scene(root, PANEL_WIDTH, PANEL_HEIGHT);

        launch(scene);
    }

    public static void main(String[] args) {
        launch(args);
    }
}
```
