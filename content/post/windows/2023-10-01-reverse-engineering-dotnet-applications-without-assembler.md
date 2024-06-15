---
id: 543
title: 'Reverse Engineering .NET Applications Without Using Assembler'
summary: 'Reverse Engineering .NET Applications Without Using Assembler'
date: '2023-10-01T21:55:05+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=543'
permalink: /2023/10/01/reverse-engineering-dotnet-applications-without-assembler/
categories:
    - Troubleshooting
---
## Reverse Engineering .NET Applications

Reverse engineering is a common practice in software development, often used for debugging, security auditing, or even academic research. While many might associate reverse engineering with delving into complex assembler code, it’s possible to reverse engineer .NET applications without touching assembler at all. In this blog post, we’ll explore why and how you can peek into .NET application code, demonstrate this with a sample application, and discuss potential solutions for protecting your code.

## Why Is It Possible to Peek Into .NET Application Code?

Introduced by Microsoft in 2002, the .NET Framework serves as a robust platform for software development. It supports multiple programming languages like C#, VB.NET, and F#, allowing developers to create a wide array of applications, ranging from web and desktop to mobile solutions.

![.NET Framework](https://cdn.porotnikov.com/media/2023/10/01214458/image.png)

When you write an application in any language compatible with the .NET Framework, the source code is first compiled into an intermediate language known as Common Intermediate Language (CIL). This language-agnostic bytecode enables cross-language interoperability.

The CIL bytecode is encapsulated in a unit called an “assembly,” which is essentially a packaged executable containing the application’s code and resources. The assembly is executed by the Common Language Runtime (CLR), a virtual machine responsible for managing .NET programs. The CLR features a Just-In-Time compiler that dynamically translates the CIL bytecode into native machine code tailored for the specific CPU architecture at runtime.

Metadata is another key component in the .NET Framework. It provides rich descriptive information about the program’s structure, such as classes, methods, and their attributes. This metadata is invaluable for various runtime services like reflection and serialization.

When a method is invoked during a .NET application’s execution, the CLR verifies its integrity and compatibility by comparing its metadata with that of the calling method. This ensures type safety and helps prevent common programming errors, thereby enhancing the robustness and security of .NET applications.

## Reflection and Decompilation Tools

The .NET Framework includes a feature known as “reflection,” which allows for the inspection of metadata at runtime. Tools like DotNetPeek and .NET Reflector leverage this feature to read the metadata and CIL code from the assembly. They then decompile the CIL back into a high-level language like C# or VB.NET, providing an approximation of the original source code. While this decompiled code may not perfectly match the original (comments are lost during compilation), it is usually very close and highly useful for understanding the application’s functionality. Let's take a look.

For demonstration purposes, I’ve created a simple C# GUI application where the user is asked to guess my favorite video game.

![Guessing Game](https://cdn.porotnikov.com/media/2023/10/01214836/image-1.png)

The source code is straightforward:

```csharp
namespace ReverseEngineeringDemo
{
    public partial class Form1 : Form
    {
        // Array of strings to guess from
        private string[] stringsToGuess = { "world of warcraft", "starcraft", "diablo", "hitman", "quake3" };
        
        public Form1()
        {
            InitializeComponent();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            // Get the user's guess from the TextBox
            string userGuess = txtGuess.Text.ToLower().Trim();

            // Check if the guess is in the array
            if (stringsToGuess.Contains(userGuess))
            {
                lblResult.Text = "Congratulations! You've guessed correctly!";
            }
            else
            {
                lblResult.Text = "Sorry, your guess was wrong. Try again.";
            }
        }
    }
}
```

To reverse engineer the app, we can use JetBrains dotPeek.

![JetBrains dotPeek](https://cdn.porotnikov.com/media/2023/10/01214903/image-2.png)

After compiling the app in “Release” mode and removing .pdb and other metadata files, we can still freely explore the code. The array of strings to guess is visible, and the application logic is exposed:

![Decompiled Code](https://cdn.porotnikov.com/media/2023/10/01214924/image-3-1024x701.jpg)

Is it possible to protect the code from being viewed this way? The answer is both yes and no. While you can deter casual reverse engineers, a dedicated individual can use tools like [Detect it Easy](https://github.com/horsicq/Detect-It-Easy) to identify obfuscation techniques and eventually [deobfuscate the code](https://github.com/NotPrab/.NET-Deobfuscator).

![Protection Mechanism](https://cdn.porotnikov.com/media/2023/10/01215047/image-3.png)

Moreover, as of the time of writing this post, there are no solid obfuscation tools for .NET 6 and .NET 7.

If you want to protect the logic of your app, consider moving it to a remote server and accessing it via an API. While this approach protects the binaries, it introduces the additional challenges of server security and cost management. But as you can see, reverse engineering .NET applications is relatively straightforward due to the framework’s architecture and the availability of decompilation tools.