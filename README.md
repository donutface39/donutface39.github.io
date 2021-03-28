# donutface39.github.io
/*
 * 5-2_MilestoneTwo_ProjectDraft.cpp
 *
 *  Created on: Aug 1, 2020
 *      Author: jennifer.dona_snhu
 */
/*Header Inclusions */
#include <iostream>
#include <GL/glew.h>
#include <GL/freeglut.h>

// GLM Math Header inclusions
#include <glm/glm.hpp>
#include <glm/gtc/matrix_transform.hpp>
#include <glm/gtc/type_ptr.hpp>

using namespace std; //Standard namspace

#define WINDOW_TITLE "Milestone Two: Project Draft" // Window Title Macro

/*Shader program Macro*/
#ifndef GLSL
#define GLSL(Version, Source) "#version " #Version "\n" #Source
#endif


/*Variable declarations for shader,window size initialization, buffer and array objects*/
GLint shaderProgram, WindowWidth = 800, WindowHeight = 600;
GLuint VBO, VAO, EBO, texture;

GLfloat cameraSpeed = 0.0005f; // Movement speed per frame

GLchar currentKey; // Will store key pressed

GLfloat lastMouseX = 400, lastMouseY = 300; // Locks mouse cursor at the center of the screen
GLfloat mouseXOffset, mouseYOffset, yaw = 0.0f, pitch = 0.0f; // mouse offset, yaw, and pitch variables
GLfloat sensitivity = 0.5f; // Used for mouse / camera rotation sensitivity
bool mouseDetected = true; // Initially true when mouse movement is detected

//Global vector declarations
glm::vec3 cameraPosition = glm::vec3(0.0f, 0.0f, 0.0f); // Initial camera position. Placed 5 units in Z
glm::vec3 CameraUpY = glm::vec3(0.0f, 1.0f, 0.0f); // Temporary y unit vector
glm::vec3 CameraForwardZ = glm::vec3(0.0f, 0.0f, -1.0f); // Temporary z unit vector
glm::vec3 front; // Temporary z unit vector for mouse

/*Function prototypes*/
void UResizeWindow(int, int);
void URenderGraphics(void);
void UCreateShader(void);
void UCreateBuffers(void);
void UMouseMove(int x, int y);

/* Vertex Shader Source Code*/
const GLchar * vertexShaderSource = GLSL(330,
	layout (location = 0) in vec3 position; // Vertex data from Vertex Attrib Pointer 0
	layout (location = 1) in vec3 color; // Color data from Vertex Attrib Pointer 1

	out vec3 mobileColor; // variable to transfer color data to the fragment shader

	//Global variables for the transform matrices
	uniform mat4 model;
	uniform mat4 view;
	uniform mat4 projection;

void main(){
	gl_Position = projection * view * model * vec4(position, 1.0f); // transforms vertices to clip coordiantes
	mobileColor = color; // references incoming color data
	}
);


/* Fragment Shader Source Code*/
const GLchar * fragmentShaderSource = GLSL(330,

		in vec3 mobileColor; // Variable to hold incoming color data from vertex shader

		out vec4 gpuColor; // Variable to pass color data to he GPU

	void main(){

		gpuColor = vec4(mobileColor, 1.0); // Sends color data to the GPU for rendering

	}
);


/*Main Program*/
int main(int argc, char* argv[])
{
	glutInit(&argc, argv);
	glutInitDisplayMode(GLUT_DEPTH | GLUT_DOUBLE | GLUT_RGBA);
	glutInitWindowSize(WindowWidth, WindowHeight);
	glutCreateWindow(WINDOW_TITLE);

	glutReshapeFunc(UResizeWindow);


	glewExperimental = GL_TRUE;
			if (glewInit() != GLEW_OK)
			{
				std::cout << "Failed to initialize GLEW" <<std:: endl;
				return -1;
			}

	UCreateShader();

	UCreateBuffers();

	//Use the Shader program
	glUseProgram(shaderProgram);

	glClearColor(0.0f, 0.0f, 0.0f, 1.0f); // Set background color

	glutDisplayFunc(URenderGraphics);

	glutPassiveMotionFunc(UMouseMove); // Detects mouse movement

	glutMainLoop();

	// Destroys Buffer object once used
	glDeleteVertexArrays(1, &VAO);
	glDeleteBuffers(1, &VBO);
	glDeleteBuffers(1, &EBO);

	return 0;
}

/* Resized the window*/
void UResizeWindow(int w, int h)
{
	WindowWidth = w;
	WindowHeight = h;
	glViewport(0, 0, WindowWidth, WindowHeight);
}


/* Renders graphics */
void URenderGraphics(void)
{

	glEnable(GL_DEPTH_TEST); // Enable z-depth

	glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); // Clears the screen

	glBindVertexArray(VAO); // Activates the Vertex Array Object before rendering and transforming them

	//*Camera Movement Logic*/
	CameraForwardZ = front; // Replaces camera forward vector with Radians normalized as a unit vector


	// Transforms the object
	glm::mat4 model;
	model = glm::translate(model, glm::vec3(0.0f, 0.0f, 0.0f)); // Place the object at the center of the viewport
	model = glm::rotate(model, 45.0f, glm::vec3(0.0f, 1.0f, 0.0f)); // Rotate the object 45 degrees on the x
	model = glm::scale(model, glm::vec3(2.0f, 2.0f, 2.0f)); // Increase the object size by a scale of 2

	// Transforms the camera
	glm::mat4 view;
	view = glm::lookAt(cameraPosition - CameraForwardZ, cameraPosition, CameraUpY);

	// Creates a perspective projection
	glm::mat4 projection;
	projection = glm::perspective(45.0f, (GLfloat)WindowWidth / (GLfloat)WindowHeight, 0.1f, 100.0f);

	// Retrieves and passes transform matrices to the shader program
	GLint modelLoc = glGetUniformLocation(shaderProgram, "model");
	GLint viewLoc = glGetUniformLocation(shaderProgram, "view");
	GLint projLoc = glGetUniformLocation(shaderProgram, "projection");

	glUniformMatrix4fv(modelLoc, 1, GL_FALSE, glm::value_ptr(model));
	glUniformMatrix4fv(viewLoc, 1, GL_FALSE, glm::value_ptr(view));
	glUniformMatrix4fv(projLoc, 1, GL_FALSE, glm::value_ptr(projection));

	glutPostRedisplay();

	// Draws the triangles
	glDrawElements(GL_TRIANGLES, 36, GL_UNSIGNED_INT, 0);

	glBindVertexArray(0); // Deactivates the Vertex Array Object

	glutPassiveMotionFunc(UMouseMove); // detects mouse movement

	glutSwapBuffers(); // Flips the back with the front buffer every frame. Similar to GL flush

}

/*Creates the Shader Program*/
void UCreateShader()
{

	// Vertex shader
	GLint vertexShader = glCreateShader(GL_VERTEX_SHADER); // Creates the Vertex shader
	glShaderSource(vertexShader, 1, &vertexShaderSource, NULL); // Attaches the Vertex shader to the source code
	glCompileShader(vertexShader); // Compiles the Vertex shader

	// Fragment shader
	GLint fragmentShader = glCreateShader(GL_FRAGMENT_SHADER); 	// Creates the Fragment shader
	glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL); // Attaches the Fragment shader to the source code
	glCompileShader(fragmentShader); // Compile the Fragment shader

	// Shader program
	shaderProgram = glCreateProgram();
	glAttachShader(shaderProgram, vertexShader); // Attach Vertex shader to the Shader program
	glAttachShader(shaderProgram, fragmentShader); // Attach Fragment shader to the Shader program
	glLinkProgram(shaderProgram); // Link Vertex and Fragment shaders to Shader program

	// Delete the Vertex and Fragment shaders once linked
	glDeleteShader(vertexShader);
	glDeleteShader(fragmentShader);

}

void UCreateBuffers()
{

	GLfloat vertices[] = {
									// Table Top		 		//Colors
									 0.5f, 0.1f,  0.0f,			1.0f, 0.0f, 0.0f,
									 0.5f, 0.0f,  0.0f,			1.0f, 0.0f, 0.0f,
									-0.5f, 0.0f,  0.0f,			1.0f, 0.0f, 0.0f,
									-0.5f, 0.1f,  0.0f,			1.0f, 0.0f, 0.0f,
									 0.5f, 0.0f, -1.0f,			1.0f, 0.0f, 0.0f,
									 0.5f, 0.1f, -1.0f,			1.0f, 0.0f, 0.0f,
									-0.5f, 0.1f, -1.0f,			1.0f, 0.0f, 0.0f,
									-0.5f, 0.0f, -1.0f,			1.0f, 0.0f, 0.0f,

									// Table Legs				//Colors
									0.5f, -1.0f, -1.0f,			1.0f, 0.0f, 0.0f,
									0.6f, -1.0f, -1.0f,			1.0f, 0.0f, 0.0f,
									0.5f, -1.0f,  0.0f,			1.0f, 0.0f, 0.0f,
									0.6f, -1.0f,  0.0f,			1.0f, 0.0f, 0.0f,
								   -0.4f,  0.0f,  0.0f,			1.0f, 0.0f, 0.0f,
								   -0.5f, -1.0f,  0.0f,			1.0f, 0.0f, 0.0f,
								   -0.4f, -1.0f,  0.0f,			1.0f, 0.0f, 0.0f,
								   -0.4f,  0.0f, -1.0f,			1.0f, 0.0f, 0.0f,
								   -0.5f, -1.0f, -1.0f,			1.0f, 0.0f, 0.0f,
								   -0.4f, -1.0f, -1.0f,			1.0f, 0.0f, 0.0f

								};

	// Index Data to Share position data
	GLuint indices[] = {
							// Table Top
							0, 1, 3,		// Triangle 1
							1, 2, 3,		// Triangle 2
							0, 1, 4,		// Triangle 3
							0, 4, 5,		// Triangle 4
							0, 5, 6,		// Triangle 5
							0, 3, 6,		// Triangle 6
							4, 5, 6,		// Triangle 7
							4, 6, 7,		// Triangle 8
							2, 3, 6,		// Triangle 9
							2, 6, 7,		// Triangle 10
							1, 4, 7,		// Triangle 12

							// Table Legs
							1, 4, 8,		// Triangle 13
							4, 8, 9,		// Triangle 14
							8, 9, 10,		// Triangle 15
							9, 10, 11,		// Triangle 16
							1, 12, 13,		// Triangle 17
							1, 4, 13,		// Triangle 18

						};

		//Generate buffer ids
		glGenVertexArrays(1, &VAO);
		glGenBuffers(1, &VBO);
		glGenBuffers(1, &EBO);

		// Activate the Vertex Array Object before binding and setting any VBOs and Vertex Attribute Pointers.
		glBindVertexArray(VAO);

		// Activate the VBO
		glBindBuffer(GL_ARRAY_BUFFER, VBO);
		glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); // Copy vertices to VBO

		// Activate the EBO
		glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
		glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW); // Copy indices to EBO

		// Set attribute pointer 0 to hold Position data
		glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6* sizeof(GLfloat), (GLvoid*)0);
		glEnableVertexAttribArray(0); // Enables vertex attribute

		// Set attribute pointer 1 to hold Position data
		glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(GLfloat), (GLvoid*)(3 * sizeof(GLfloat)));

		glBindVertexArray(0); // Deactivates the VAO which is good practice
}
/*Implements the UMouseMove function*/
void UMouseMove(int x, int y)
{
	// Immediately replaces center locked coordinates with new mouse coordinates
	if(mouseDetected)
		{
			lastMouseX = x;
			lastMouseY = y;
			mouseDetected = false;
		}

	// Gets the direction the mouse was moved in x and y
	mouseXOffset = x - lastMouseX;
	mouseYOffset = lastMouseY - y; // Inverted y

	//Updates with new mouse coordinates
	lastMouseX = x;
	lastMouseY = y;

	//Applies sensitivity to mouse direction
	mouseXOffset *= sensitivity;
	mouseYOffset *= sensitivity;

	//Accumulates the yaw and pitch variables
	yaw += mouseXOffset;
	pitch += mouseYOffset;

	//Orbits around the center
	front.x = 10.0f * cos(yaw);
	front.y = 10.0f * sin(pitch);
	front.z = sin(yaw) * cos(pitch) * 10.0f;
}
