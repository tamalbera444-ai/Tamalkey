# Tamalkey
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "AiKeyboard"
include(":app")
plugins {
    id("com.android.application") version "8.5.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.24" apply false
}
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
android.nonTransitiveRClass=true
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.9-bin.zip
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.example.aikeyboard"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.aikeyboard"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "0.1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.8.1")
    <?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="false"
        android:label="@string/app_name"
        android:theme="@android:style/Theme.Material.Light.DarkActionBar">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".AiKeyboardService"
            android:label="@string/app_name"
            android:permission="android.permission.BIND_INPUT_METHOD"
            android:exported="true">
            <intent-filter>
                <action android:name="android.view.InputMethod" />
            </intent-filter>
            <meta-data
                android:name="android.view.im"
                android:resource="@xml/method" />
        </service>
    </application>
</manifest>
}
<?xml version="1.0" encoding="utf-8"?>
<input-method xmlns:android="http://schemas.android.com/apk/res/android"
    android:settingsActivity="com.example.aikeyboard.MainActivity" />
    <?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">AI Keyboard</string>
    <string name="setup_instructions">Step 1: tap Enable keyboard and turn on AI Keyboard.\n\nStep 2: tap Choose keyboard and select AI Keyboard.\n\nStep 3: open any text field and type: how r u ?fix</string>
    <string name="btn_enable">Enable keyboard</string>
    <string name="btn_pick">Choose keyboard</string>
</resources>
package com.example.aikeyboard

import android.app.Activity
import android.content.Context
import android.content.Intent
import android.os.Bundle
import android.provider.Settings
import android.view.inputmethod.InputMethodManager
import android.widget.Button
import android.widget.LinearLayout
import android.widget.TextView

/** Tiny setup screen: opens Android's keyboard settings and the keyboard picker. */
class MainActivity : Activity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val pad = (24 * resources.displayMetrics.density).toInt()

        val root = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            setPadding(pad, pad, pad, pad)
        }

        root.addView(TextView(this).apply {
            text = getString(R.string.setup_instructions)
            textSize = 16f
        })

        root.addView(Button(this).apply {
            text = getString(R.string.btn_enable)
            setOnClickListener {
                startActivity(Intent(Settings.ACTION_INPUT_METHOD_SETTINGS))
            }
        })

        root.addView(Button(this).apply {
            text = getString(R.string.btn_pick)
            setOnClickListener {
                val imm = getSystemService(Context.INPUT_METHOD_SERVICE) as InputMethodManager
                imm.showInputMethodPicker()
            }
        })

        setContentView(root)
    }
}package com.example.aikeyboard

import android.content.Context
import android.widget.Button
import android.widget.LinearLayout

/**
 * Minimal temporary QWERTY so you can test triggers on a real phone.
 * Replace it with the Gboard-style view later.
 */
class SimpleKeyboardView(
    context: Context,
    private val onText: (String) -> Unit,
    private val onBackspace: () -> Unit,
    private val onEnter: () -> Unit
) : LinearLayout(context) {

    private val dp = context.resources.displayMetrics.density
    private var shift = false

    init {
        orientation = LinearLayout.VERTICAL
        setBackgroundColor(0xFF202124.toInt())
        setPadding(dpi(4), dpi(4), dpi(4), dpi(4))

        row { "1234567890".forEach { c -> addView(key(c.toString()) { onText(c.toString()) }) } }
        row { "qwertyuiop".forEach { c -> addView(letterKey(c)) } }
        row { "asdfghjkl".forEach { c -> addView(letterKey(c)) } }
        row {
            addView(key("⇧", 1.5f) { shift = !shift })
            "zxcvbnm".forEach { c -> addView(letterKey(c)) }
            addView(key("⌫", 1.5f) { onBackspace() })
        }
        row {
            "?,:-".forEach { c -> addView(key(c.toString()) { onText(c.toString()) }) }
            addView(key("space", 4f) { onText(" ") })
            addView(key(".") { onText(".") })
            addView(key("⏎", 1.5f) { onEnter() })
        }
    }

    private fun dpi(v: Int): Int = (v * dp).toInt()

    private fun row(fill: LinearLayout.() -> Unit) {
        val r = LinearLayout(context).apply {
            orientation = LinearLayout.HORIZONTAL
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT, dpi(50)
            )
            fill()
        }
        addView(r)
    }

    private fun letterKey(c: Char): Button = key(c.toString()) {
        onText(if (shift) c.uppercase() else c.toString())
        shift = false
    }

    private fun key(label: String, weight: Float = 1f, action: () -> Unit): Button =
        Button(context).apply {
            text = label
            isAllCaps = false
            textSize = 16f
            setTextColor(0xFFFFFFFF.toInt())
            setBackgroundColor(0xFF3C4043.toInt())
            minWidth = 0
            minimumWidth = 0
            setPadding(0, 0, 0, 0)
            layoutParams = LinearLayout.LayoutParams(
                0, LinearLayout.LayoutParams.MATCH_PARENT, weight
            ).apply { setMargins(dpi(2), dpi(2), dpi(2), dpi(2)) }
            setOnClickListener { action() }
        }
}package com.example.aikeyboard

import android.inputmethodservice.InputMethodService
import android.text.InputType
import android.view.KeyEvent
import android.view.View
import android.view.inputmethod.EditorInfo
import android.widget.Toast
import kotlinx.coroutines.*

class AiKeyboardService : InputMethodService() {

    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main.immediate)
    private var debounceJob: Job? = null
    private var workJob: Job? = null
    private var session = 0                    // changes whenever the target field changes
    private lateinit var ai: AiProvider

    override fun onCreate() {
        super.onCreate()
        ai = FakeAiProvider()                  // TODO: swap for the real multi-key client
    }

    override fun onCreateInputView(): View =
        SimpleKeyboardView(
            this,
            onText = { currentInputConnection?.commitText(it, 1) },
            onBackspace = { sendDownUpKeyEvents(KeyEvent.KEYCODE_DEL) },
            onEnter = { sendDownUpKeyEvents(KeyEvent.KEYCODE_ENTER) }
        )

    // ---- Field lifecycle ------------------------------------------------------

    override fun onStartInput(info: EditorInfo?, restarting: Boolean) {
        super.onStartInput(info, restarting)
        if (!restarting) session++             // a genuinely new field
    }

    override fun onFinishInput() {
        super.onFinishInput()
        debounceJob?.cancel()
        workJob?.cancel()
    }

    override fun onDestroy() {
        scope.cancel()
        super.onDestroy()
    }

    // ---- Trigger watching -----------------------------------------------------
    // Fires after any text change (our keys, swipe, voice, suggestions).

    override fun onUpdateSelection(
        oldSelStart: Int, oldSelEnd: Int, newSelStart: Int, newSelEnd: Int,
        candidatesStart: Int, candidatesEnd: Int
    ) {
        super.onUpdateSelection(
            oldSelStart, oldSelEnd, newSelStart, newSelEnd, candidatesStart, candidatesEnd
        )
        if (workJob?.isActive == true) return  // spinner updates also land here
        debounceJob?.cancel()
        debounceJob = scope.launch {
            delay(DEBOUNCE_MS)                 // lets "?fix" finish before "?fixup"
            checkForTrigger()
        }
    }

    private fun checkForTrigger() {
        val ic = currentInputConnection ?: return
        val info = currentInputEditorInfo ?: return
        if (isSensitiveField(info)) return     // never send passwords to an AI

        val before = ic.getTextBeforeCursor(MAX_CHARS, 0)?.toString() ?: return
        val match = TriggerDetector.detect(before) ?: return
        val after = ic.getTextAfterCursor(MAX_CHARS, 0)?.toString().orEmpty()
        val input = CommandInput(match.before, after, match.arg)

        if (match.command.needsText && input.text.isBlank()) return
        execute(match, input)
    }

    // ---- Run command, spinner, replace ----------------------------------------

    private fun execute(match: TriggerMatch, input: CommandInput) {
        val ic = currentInputConnection ?: return
        val mySession = session

        // Remove only the "?fix" part; the user's text stays safe until we have a result.
        ic.deleteSurroundingText(match.triggerText.length, 0)

        workJob = scope.launch {
            val spinner = launch { spin() }
            val result: String? = try {
                withContext(Dispatchers.IO) { match.command.run(input, ai) }
            } catch (e: CancellationException) {
                throw e
            } catch (e: Exception) {
                null
            }
            spinner.cancelAndJoin()
            if (mySession != session) return@launch    // user moved to another field

            val out = currentInputConnection ?: return@launch
            out.beginBatchEdit()
            out.setComposingText("", 1)                 // clear spinner
            out.finishComposingText()

            val unchanged =
                out.getTextBeforeCursor(match.before.length, 0)?.toString() == match.before
            if (result.isNullOrEmpty() || !unchanged) {
                out.commitText(match.triggerText, 1)    // restore what the user typed
                if (result == null) toast("AI request failed")
            } else {
                out.deleteSurroundingText(match.before.length, input.after.length)
                out.commitText(result, 1)
            }
            out.endBatchEdit()
        }
    }

    /** Inline braille spinner, drawn as composing text at the cursor. */
    private suspend fun spin() {
        val frames = "⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏"
        var i = 0
        while (true) {
            currentInputConnection?.setComposingText(frames[i++ % frames.length].toString(), 1)
            delay(80)
        }
    }

    // ---- Helpers --------------------------------------------------------------

    private fun isSensitiveField(info: EditorInfo): Boolean {
        val cls = info.inputType and InputType.TYPE_MASK_CLASS
        val variation = info.inputType and InputType.TYPE_MASK_VARIATION
        val password = when (cls) {
            InputType.TYPE_CLASS_TEXT -> variation == InputType.TYPE_TEXT_VARIATION_PASSWORD ||
                variation == InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD ||
                variation == InputType.TYPE_TEXT_VARIATION_WEB_PASSWORD
            InputType.TYPE_CLASS_NUMBER -> variation == InputType.TYPE_NUMBER_VARIATION_PASSWORD
            else -> false
        }
        val incognito = info.imeOptions and EditorInfo.IME_FLAG_NO_PERSONALIZED_LEARNING != 0
        return password || incognito
    }

    private fun toast(msg: String) = Toast.makeText(this, msg, Toast.LENGTH_SHORT).show()

    companion object {
        private const val DEBOUNCE_MS = 400L
        private const val MAX_CHARS = 4000
    }
}package com.example.aikeyboard

data class TriggerMatch(
    val command: Command,
    val before: String,       // text before the trigger, e.g. "how r u "
    val triggerText: String,  // the raw trigger, e.g. "?fix" or "?translate:fr"
    val arg: String?          // "fr" in "?translate:fr"
)

object TriggerDetector {

    // "?word" or "?word:arg" at the very end of the text before the cursor.
    private val TAIL = Regex("""\?([A-Za-z]+)(?::([A-Za-z\-]{2,12}))?$""")

    /** Returns a match only if the trailing "?word" is a registered command. */
    fun detect(textBeforeCursor: String): TriggerMatch? {
        val m = TAIL.find(textBeforeCursor) ?: return null
        val command = CommandRegistry.find(m.groupValues[1]) ?: return null
        val start = m.range.first
        return TriggerMatch(
            command = command,
            before = textBeforeCursor.substring(0, start),
            triggerText = textBeforeCursor.substring(start),
            arg = m.groups[2]?.value
        )
    }
}package com.example.aikeyboard

import kotlinx.coroutines.delay

/** Talks to an AI model. Next step: a real client with multi-key rotation. */
interface AiProvider {
    suspend fun complete(systemPrompt: String, userText: String): String
}

/** Placeholder so you can test the whole flow without an API key. */
class FakeAiProvider : AiProvider {
    override suspend fun complete(systemPrompt: String, userText: String): String {
        delay(1500) // pretend to think, so you can see the spinner
        return "[AI] ${userText.trim()}"
    }
}

/** What the user typed around the trigger. */
class CommandInput(val before: String, val after: String, val arg: String?) {
    val text: String get() = before + after
}

/** One "?trigger". Add new buttons/commands by implementing this. */
interface Command {
    val trigger: String                       // without the leading '?'
    val needsText: Boolean get() = true       // false = works on an empty field
    suspend fun run(input: CommandInput, ai: AiProvider): String
}

/** Sends the text to the AI with a prompt. {arg} is filled from "?translate:fr". */
class AiCommand(override val trigger: String, private val prompt: String) : Command {
    override suspend fun run(input: CommandInput, ai: AiProvider): String =
        ai.complete(
            prompt.replace("{arg}", input.arg ?: "") +
                " Reply with only the resulting text, no quotes or explanations.",
            input.text.trim()
        ).trim()
}

/** Offline snippet: "?sig" is replaced in place, no network. */
class SnippetCommand(override val trigger: String, private val snippet: String) : Command {
    override val needsText: Boolean = false
    override suspend fun run(input: CommandInput, ai: AiProvider): String =
        input.before + snippet + input.after
}

object CommandRegistry {
    private val map = LinkedHashMap<String, Command>()

    fun register(c: Command) { map[c.trigger.lowercase()] = c }
    fun find(trigger: String): Command? = map[trigger.lowercase()]

    init {
        register(AiCommand("fix", "Fix grammar, spelling and punctuation. Keep the meaning and tone."))
        register(AiCommand("improve", "Improve clarity and readability."))
        register(AiCommand("shorten", "Shorten the text while keeping its meaning."))
        register(AiCommand("expand", "Expand the text with more detail."))
        register(AiCommand("formal", "Rewrite in a professional tone."))
        register(AiCommand("casual", "Rewrite in a friendly, casual tone."))
        register(AiCommand("emoji", "Add relevant emojis to the text."))
        register(AiCommand("translate", "Translate the text to {arg}."))
        register(SnippetCommand("sig", "\n\nBest regards,\nYour Name"))
    }
}name: Build debug APK

on:
  push:
    branches: [main, master]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"

      - name: Set up Gradle 8.9
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: "8.9"

      - name: Build debug APK
        run: gradle assembleDebug --stacktrace

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: AiKeyboard-debug-apk
          path: app/build/outputs/apk/debug/app-debug.apk
          if-no-files-found: error
