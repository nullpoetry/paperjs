# paperjs
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Asemic Paper.js Geodesic Rainbow Art</title>
    <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/paper.js/0.12.15/paper-full.min.js"></script>
    <style>
        body {
            margin: 0;
            overflow: hidden; /* Hide scrollbars if canvas is full screen */
            background-color: #333; /* Dark background for contrast */
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh; /* Make body at least full viewport height */
        }
        canvas {
            background-color: #1a1a1a; /* Darker background for the canvas itself */
            /* You can set a fixed size or make it responsive */
            width: 80vw; /* 80% of viewport width */
            height: 80vh; /* 80% of viewport height */
            max-width: 800px; /* Max width to prevent it from getting too large */
            max-height: 600px; /* Max height */
            border-radius: 10px; /* Slightly rounded corners */
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5); /* Subtle shadow */
        }
    </style>
</head>
<body>

    <canvas id="myCanvas" resize></canvas>

    <script type="text/paperscript" canvas="myCanvas">
        // This script block uses Paper.js, tied to the 'myCanvas' element
        // The 'resize' attribute on the canvas ensures it fits the window automatically

        // Function to generate a simple rainbow color based on an index
        function getRainbowColor(index, total, saturation = 100, lightness = 70) {
            let hue = (index / total) * 360;
            return `hsl(${hue}, ${saturation}%, ${lightness}%)`;
        }

        // --- Geodesic/Origami Simulation Logic ---
        // This is a simplified representation to get something visual running.
        // A true geodesic sphere would involve complex math (e.g., icosahedron subdivision).
        // A true origami simulation would involve manipulating path segments to simulate folds and creases.

        let numSegments = 18; // Number of "faces" in our circular geodesic-like structure
        let initialRadius = Math.min(view.size.width, view.size.height) * 0.35; // Radius based on canvas size
        let center = view.center;
        let paths = []; // Store our shapes to manipulate them later

        // Create initial shapes (simplified geodesic faces)
        for (let i = 0; i < numSegments; i++) {
            let angle = (i / numSegments) * 360;
            let nextAngle = ((i + 1) / numSegments) * 360;

            let point1 = center.add(new Point(Math.cos(angle * Math.PI / 180) * initialRadius, Math.sin(angle * Math.PI / 180) * initialRadius));
            let point2 = center.add(new Point(Math.cos(nextAngle * Math.PI / 180) * initialRadius, Math.sin(nextAngle * Math.PI / 180) * initialRadius));

            // Create a basic "triangle" representing a simplified geodesic face
            let path = new Path();
            path.add(center);
            path.add(point1);
            path.add(point2);
            path.closed = true;

            // Apply rainbow color
            path.fillColor = getRainbowColor(i, numSegments);
            path.strokeColor = 'rgba(255, 255, 255, 0.2)'; // Subtle white stroke
            path.strokeWidth = 0.5;

            // --- Asemic / Origami-like detail (simple crease simulation) ---
            // Add a line across the triangle to simulate a crease or fold
            let midPoint = point1.add(point2).divide(2);
            let creaseLine = new Path.Line(center, midPoint);
            creaseLine.strokeColor = new Color(0, 0, 0, 0.3); // Darker line for the crease
            creaseLine.strokeWidth = 1;
            creaseLine.strokeCap = 'round'; // Make ends rounded

            // Group the path and its crease line so they move together
            let group = new Group([path, creaseLine]);
            paths.push(group);
        }

        // --- Animation Logic ---
        view.onFrame = function(event) {
            // Animate the colors
            for (let i = 0; i < paths.length; i++) {
                let group = paths[i];
                let path = group.children[0]; // Get the main shape from the group

                // Shift hue over time for a continuous rainbow effect
                // Adjust the '0.5' to change the speed of color animation
                let animatedHue = (path.fillColor.hue + event.delta * 20) % 360;
                path.fillColor.hue = animatedHue;

                // Simple 'breathing' scale animation for origami effect
                // Adjust the '0.05' and '0.5' for intensity and speed
                let scaleFactor = 1 + Math.sin(event.time * 2 + i * 0.5) * 0.05;
                group.scale(scaleFactor / group.scaling.x, center); // Apply scaling from the center
            }

            // Optional: Rotate the entire collection of shapes slightly
            // project.activeLayer.rotate(0.1, center);
        };

        // --- Responsiveness on Canvas Resize ---
        view.onResize = function(event) {
            // Recalculate center and radius if the canvas changes size
            center = view.center;
            initialRadius = Math.min(view.size.width, view.size.height) * 0.35;

            // Update positions of existing shapes
            for (let i = 0; i < numSegments; i++) {
                let angle = (i / numSegments) * 360;
                let nextAngle = ((i + 1) / numSegments) * 360;

                let group = paths[i];
                let path = group.children[0];
                let creaseLine = group.children[1];

                let point1 = center.add(new Point(Math.cos(angle * Math
