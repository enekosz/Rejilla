<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Grafeno y Mercurio Interactivo</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/examples/js/controls/OrbitControls.js"></script>
    <style>
        body { margin: 0; background-color: black; text-align: center; font-family: Arial, sans-serif; }
        canvas { display: block; }
        .controls {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 10;
        }
        button {
            background-color: #222;
            color: white;
            border: 1px solid white;
            padding: 10px 15px;
            margin: 5px;
            cursor: pointer;
        }
        button:hover { background-color: #444; }
    </style>
</head>
<body>

<div class="controls">
    <button onclick="startAnimation()">Iniciar</button>
    <button onclick="pauseAnimation()">Pausar</button>
    <button onclick="resetAnimation()">Reiniciar</button>
</div>

<script>
    // Configuración de la escena
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 100);
    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // Controles de cámara con el ratón
    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    // Crear la rejilla de grafeno (estructura hexagonal)
    const grafenoMaterial = new THREE.LineBasicMaterial({ color: 0x888888 });
    const grafenoGeometry = new THREE.BufferGeometry();
    let vertices = [];
    const hexSize = 1;

    for (let i = -5; i <= 5; i++) {
        for (let j = -5; j <= 5; j++) {
            const xOffset = i * 1.5 * hexSize;
            const yOffset = j * Math.sqrt(3) * hexSize + (i % 2 === 0 ? 0 : Math.sqrt(3) / 2);
            let hexagon = [
                [xOffset, yOffset, 0], [xOffset + hexSize, yOffset, 0],
                [xOffset + 1.5 * hexSize, yOffset + Math.sqrt(3) / 2 * hexSize, 0],
                [xOffset + hexSize, yOffset + Math.sqrt(3) * hexSize, 0],
                [xOffset, yOffset + Math.sqrt(3) * hexSize, 0],
                [xOffset - 0.5 * hexSize, yOffset + Math.sqrt(3) / 2 * hexSize, 0],
                [xOffset, yOffset, 0]
            ];
            hexagon.forEach(v => vertices.push(...v));
        }
    }

    grafenoGeometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
    const grafeno = new THREE.LineSegments(grafenoGeometry, grafenoMaterial);
    scene.add(grafeno);

    // Crear la esfera de mercurio
    const mercurioGeometry = new THREE.SphereGeometry(0.5, 32, 32);
    const mercurioMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });
    const mercurio = new THREE.Mesh(mercurioGeometry, mercurioMaterial);
    mercurio.position.set(2.5, 2.5, 5);
    scene.add(mercurio);

    // Posicionar la cámara
    camera.position.set(0, -10, 5);
    camera.lookAt(0, 0, 0);

    // Variables de animación
    let velocity = -0.02;
    let animationRunning = false;
    let bounceFactor = 0.6; // Factor de rebote
    let gravity = -0.02;

    function animate() {
        if (animationRunning) {
            // Simular caída con rebote
            if (mercurio.position.z > 0.5) {
                velocity += gravity; // Aceleración por gravedad
                mercurio.position.z += velocity;
            } else {
                velocity = -velocity * bounceFactor; // Rebote
                if (Math.abs(velocity) < 0.01) {
                    velocity = 0; // Detener rebote pequeño
                }
            }
        }

        controls.update();
        renderer.render(scene, camera);
        requestAnimationFrame(animate);
    }

    // Funciones de control
    function startAnimation() { animationRunning = true; }
    function pauseAnimation() { animationRunning = false; }
    function resetAnimation() {
        animationRunning = false;
        mercurio.position.z = 5;
        velocity = -0.02;
    }

    animate(); // Iniciar animación
</script>

</body>
</html>
