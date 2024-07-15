# MathTutor

MathTutor is a SwiftUI application that helps users practice basic arithmetic operations (addition, subtraction, and multiplication) using a fun and interactive interface. The app uses emoji representations of numbers to make learning more engaging and enjoyable for users.

## Features

- **Interactive Arithmetic Practice**: Users can practice addition, subtraction, and multiplication.
- **Emoji Representation**: Numbers are represented using emojis to make learning fun.
- **Sound Effects**: Correct and incorrect answers trigger sound effects to provide feedback.

## Screenshots

![MathTutor](screenshots/demo.gif)

## Requirements

- iOS 15.0+
- Xcode 13.0+
- SwiftUI

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/dobson980/MathTutor.git
   ```
2. Open the project in Xcode:
   ```bash
   cd MathTutor
   open MathTutor.xcodeproj
   ```

## Usage

### Main View
The ContentView is the main view of the app where users can solve arithmetic problems by inputting their answer and receiving immediate feedback.

## Code Overview

### MathTutorApp
MathTutorApp is the entry point of the app.
```swift
import SwiftUI

@main
struct MathTutorApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### ContentView
ContentView sets up the main view with arithmetic problems, emoji representations, and input fields for users to solve the problems.

```swift
import SwiftUI
import AVFAudio

struct ContentView: View {
    @State private var answer = ""
    @State private var addend1 = 0
    @State private var addend2 = 0
    @State private var addend1Emoji = ""
    @State private var addend2Emoji = ""
    @State private var addend1Display = ""
    @State private var addednd2Display = ""
    @State private var selectedOperator = ""
    @State private var correctAnswer = 0
    @State private var guessedCorrect = false
    @State private var showingResults = false
    @State private var audiPlayer: AVAudioPlayer?
    @FocusState private var isFocused: Bool
    
    private let operators = ["+", "-", "*"]
    private let emojis = [
        "🍕", "🍎", "🍏", "🐵", "👽",
        "🧠", "🧜🏽‍♀️", "🧙🏿‍♂️", "🥷", "🐶",
        "🐹", "🐣", "🦄", "🐝", "🦉",
        "🦋", "🦖", "🐙", "🦞", "🐟",
        "🦔", "🐲", "🌻", "🌍", "🌈",
        "🍔", "🌮", "🍦", "🍩", "🍪"
    ]
    
    var body: some View {
        VStack(spacing: 0) {
            
            VStack(spacing: 0) {
                Text(addend1Display)
                    .frame(height: 150)
                Text(selectedOperator)
                Text(addednd2Display)
                    .frame(height: 150)

            }
            .font(.system(size: 60))
            .multilineTextAlignment(.center)
            .frame(maxWidth: .infinity)
            .padding(.horizontal)
                        
            Text("\(addend1) \(selectedOperator) \(addend2) =")
                .font(.title)
            
            TextField("", text: $answer)
                .frame(width: 50, height: 50)
                .multilineTextAlignment(.center)
                .font(.title2)
                .overlay {
                    RoundedRectangle(cornerRadius: 5)
                        .stroke(.gray, lineWidth: 2)
                }
                .padding(5)
                .keyboardType(.numberPad)
                .focused($isFocused)
                .onChange(of: answer) {
                    // Filter out any non-digit characters
                    let filtered = answer.filter { "0123456789".contains($0) }
                    // If the filtered string has more than 2 characters, limit it to 2 characters
                    if filtered.count > 3 {
                        // Keep only the first 2 characters
                        answer = String(filtered.prefix(3))
                    } else {
                        // Update the text with the filtered string
                        answer = filtered
                    }
                }
            
            Button("Guess") {
                isFocused = false
                
                print("correct answer: \(correctAnswer)")
                print("answer: \(answer)")
                if correctAnswer == Int(answer) {
                    playSound(soundName: "correct")
                    guessedCorrect = true
                    showingResults = true
                } else {
                    playSound(soundName: "wrong")
                    guessedCorrect = false
                    showingResults = true
                }
                
            }
            .buttonStyle(.borderedProminent)
            .disabled(answer.isEmpty || showingResults)
            
            Spacer()
            Spacer()
            
            Group {
                
                if showingResults {
                    if guessedCorrect {
                        Text("Correct")
                            .font(.system(size: 50))
                            .frame(height: 120)
                            .multilineTextAlignment(.center)
                            .foregroundColor(.green)
                    } else {
                        VStack {
                            Text("Wrong!")
                            Text("Answer was: \(correctAnswer)")
                        }
                        .font(.system(size: 40))
                        .multilineTextAlignment(.center)
                        .frame(height: 120)
                        .foregroundColor(.red)
                    }
                    
                    Button("Play Again?") {
                        roundSetup()
                    }
                    .buttonStyle(.borderedProminent)
                }
                
            }
        }
        .padding()
        .onAppear() {
            roundSetup()
        }
    }
    
    func generateEmojiArray(quanity: Int, emoji: String) -> String {
        var output = ""
        
        for _ in 1...quanity {
            output += emoji
        }
        
        return output
    }
    
    func doMath(addend1: Int, addend2: Int, selectedOperator: String) -> Int {
        switch selectedOperator {
        case("+"):
            return addend1 + addend2
        case("-"):
            return addend1 - addend2
        case("*"):
            return addend1 * addend2
        default: return 0
        }
    }
    
    func roundSetup() {
        answer = ""
        guessedCorrect = false
        showingResults = false
        
        addend1 = Int.random(in: 1...10)
        addend2 = Int.random(in: 1...10)
        selectedOperator = operators.randomElement() ?? "+"
        repeat {
            addend1Emoji = emojis.randomElement() ?? "❓"
            addend2Emoji = emojis.randomElement() ?? "❓"
        } while addend1Emoji == addend2Emoji
        
        addend1Display = generateEmojiArray(quanity: addend1, emoji: addend1Emoji)
        addednd2Display = generateEmojiArray(quanity: addend2, emoji: addend2Emoji)
        
        correctAnswer = doMath(addend1: addend1, addend2: addend2, selectedOperator: selectedOperator)
    }
    
    func playSound(soundName: String) {
        guard let soundFile = NSDataAsset(name: soundName) else {
            print("Could not read sound: \(soundName)")
            return
        }
        do {
            audiPlayer = try AVAudioPlayer(data: soundFile.data)
            audiPlayer?.play()
        } catch {
            print("Error: \(error.localizedDescription) when creating audio player")
        }
    }
}

#Preview {
    ContentView()
}
```

## License

This project is licensed under the MIT License. See the [LICENSE](license) file for details.
