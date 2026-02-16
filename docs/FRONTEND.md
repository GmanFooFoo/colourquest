# ColorQuest Animate — AI Frontend Instructions

## Design Philosophy

**One-page, kid-friendly creative tool** that guides users through three magical steps without overwhelming them. The UI should feel playful, responsive, and forgiving.

**Core Principle**: Each step (Generate → Upload → Animate) is its own contained experience with visual feedback at every stage.

---

## Page Layout Architecture

### 1. Page Structure (Single Canvas)

```
┌────────────────────────────────────────────────────────────────┐
│ HEADER: 🎨 ColorQuest Animate - "Turn imagination into magic"  │
├──────────────────────┬─────────────────────────────────────────┤
│                      │                                         │
│   SIDEBAR            │        MAIN CONTENT AREA                │
│   (Navigation +      │        (Step-specific UI)               │
│    Input Controls)   │                                         │
│                      │        Step 1: Generate                 │
│   ┌─────────────┐    │        Step 2: Upload                   │
│   │ 🎬 Generate │    │        Step 3: Animate                  │
│   └─────────────┘    │                                         │
│   ┌─────────────┐    │                                         │
│   │ 📤 Upload   │    │        [Use session_state to            │
│   └─────────────┘    │         track active step]              │
│   ┌─────────────┐    │                                         │
│   │ ✨ Animate  │    │                                         │
│   └─────────────┘    │                                         │
│                      │                                         │
│   [State Indicator]  │        [Download / Share Buttons]       │
└──────────────────────┴─────────────────────────────────────────┘
```

### 2. Sidebar (Navigation & Prompt Input)

**Role**: Single source of truth for user input; shows progress.

```python
with st.sidebar:
    st.header("🪄 ColorQuest Control Panel")
    
    # Step Indicator (Visual Progress)
    st.progress(current_step / 3)
    st.write(f"Step {current_step} of 3: {['Generate', 'Upload', 'Animate'][current_step-1]}")
    
    # Step 1: Generate
    if st.session_state.step >= 1:
        st.subheader("🎬 Step 1: Generate")
        prompt = st.text_input(
            "What do you want to color?",
            placeholder="e.g., 'cute dragon flying'",
            max_chars=100,
            help="Use 1-2 objects for best results!"
        )
        col1, col2 = st.columns(2)
        with col1:
            if st.button("✨ Generate Magic", use_container_width=True):
                st.session_state.step = 1
                generate_coloring_page(prompt)
        with col2:
            if st.button("🎲 Random", use_container_width=True):
                generate_random()
    
    st.divider()
    
    # Step 2: Upload
    if st.session_state.colored_image is not None:
        st.subheader("📤 Step 2: Upload")
        st.write("Got your colored version? Upload it here!")
        uploaded_file = st.file_uploader(
            "Upload your colored artwork",
            type=["jpg", "jpeg", "png"],
            help="Take a photo with your phone or upload a file"
        )
        if uploaded_file:
            st.session_state.step = 2
            process_colored_image(uploaded_file)
        
        st.divider()
    
    # Step 3: Animate
    if st.session_state.upload_image is not None:
        st.subheader("✨ Step 3: Animate")
        st.write("Ready to bring it to life?")
        animation_style = st.radio(
            "Choose an animation style:",
            ["🤗 Wiggle", "💫 Sparkle", "🌀 Spin", "👋 Wave"],
            index=0
        )
        if st.button("🎬 Magic Animate!", use_container_width=True):
            st.session_state.step = 3
            create_animation(st.session_state.upload_image, animation_style)
    
    st.divider()
    st.caption("🦖✨ Made with ❤️ for creativity")
```

### 3. Main Content Area (Step-Specific UI)

Each step displays relevant content with large, tap-friendly elements.

#### **Step 1: Generation Display**

```python
st.header("🎨 Your Coloring Page is Ready!")

if st.session_state.generated_image:
    col1, col2 = st.columns([2, 1])
    
    with col1:
        # Large preview (2x main column width)
        st.image(st.session_state.generated_image, use_column_width=True)
        st.success("✅ Your page is ready to color!")
    
    with col2:
        st.subheader("📥 Download")
        # Convert image to PNG bytes for download
        img_bytes = image_to_bytes(st.session_state.generated_image)
        st.download_button(
            label="📥 Download PNG",
            data=img_bytes,
            file_name="colorquest_page.png",
            mime="image/png",
            use_container_width=True
        )
        st.info("""
        💡 **Tips for coloring:**
        • Use bold colors
        • Color inside the lines
        • Try markers or colored pencils!
        """)

elif st.session_state.generating:
    st.info("🎨 Generating your coloring page... This is magic in the making!")
    st.progress(0, text="Creating thick outlines...")
    time.sleep(0.5)  # Simulate progress
else:
    st.write("👈 Enter a prompt in the sidebar to get started!")
```

#### **Step 2: Upload Display**

```python
st.header("📤 Show Me Your Masterpiece!")

col1, col2 = st.columns(2)

with col1:
    st.subheader("Original (Reference)")
    st.image(st.session_state.generated_image, use_column_width=True)
    st.caption("Your coloring page")

with col2:
    st.subheader("Your Colored Version")
    if st.session_state.upload_image:
        st.image(st.session_state.upload_image, use_column_width=True)
        st.success("🎉 Great job on the colors!")
        
        # Generate color quality feedback
        feedback = analyze_coloring_quality(
            st.session_state.generated_image,
            st.session_state.upload_image
        )
        if feedback["coverage"] > 0.8:
            st.balloons()
            st.write(f"✨ **{feedback['message']}** You covered 95% of the page!")
        else:
            st.warning("💡 Try coloring more areas for a better animation!")
    else:
        st.info("📱 Upload a photo of your colored version using the sidebar")
```

#### **Step 3: Animation Preview & Export**

```python
st.header("✨ Watch It Come Alive!")

if st.session_state.animated_gif:
    # GIF preview (centered, large)
    col_center = st.columns([0.5, 1, 0.5])[1]
    with col_center:
        st.image(st.session_state.animated_gif, use_column_width=True)
        st.success("🎬 Your animation is ready!")
    
    # Export options (full width, big buttons)
    col_dl1, col_dl2 = st.columns(2)
    
    with col_dl1:
        gif_bytes = get_gif_bytes(st.session_state.animated_gif)
        st.download_button(
            label="📥 Download GIF",
            data=gif_bytes,
            file_name="colorquest_animation.gif",
            mime="image/gif",
            use_container_width=True
        )
    
    with col_dl2:
        st.button(
            label="📤 Share (Coming Soon!)",
            disabled=True,
            use_container_width=True
        )
    
    st.divider()
    st.write("**Want to create another?** Reload the page or use the sidebar!")
else:
    st.info("👈 Select an animation style in the sidebar and press 'Magic Animate!'")
```

---

## UI/UX Patterns & Best Practices

### 1. Kid-Friendly Design Rules

- **Big buttons**: Always use `use_container_width=True` for primary actions
- **Emojis everywhere**: Use as visual anchors (🎨, ✨, 📤, 🎬)
- **Clear feedback**: Show progress spinners, success messages, not errors (reframe as "let's try again")
- **Forgiving UX**: No confirmation dialogs; undo/back buttons; state preserved in `st.session_state`
- **Color scheme**: Bright, primary colors (blues, greens, purples); avoid reds for errors (use orange/yellow instead)

### 2. Session State Management

Keep all multi-step data in `st.session_state`:

```python
# Initialize session state (run once at app start)
if "step" not in st.session_state:
    st.session_state.step = 0
    st.session_state.generated_image = None
    st.session_state.generated_image_original = None  # Store original for comparison
    st.session_state.upload_image = None
    st.session_state.animated_gif = None
    st.session_state.generating = False
    st.session_state.animation_style = "Wiggle"
    st.session_state.last_prompt = ""
```

**Key Rule**: Only advance `st.session_state.step` after successful operation completion.

### 3. Progress Indicators & User Feedback

**For each long-running operation:**

```python
def generate_coloring_page(prompt):
    st.session_state.generating = True
    
    with st.spinner("🎨 Creating your coloring page..."):
        try:
            # Step 1: Validate prompt
            if len(prompt) < 3:
                st.error("💭 Tell me more! Use at least 3 words (e.g., 'blue dragon')")
                st.session_state.generating = False
                return
            
            # Step 2: Wrap prompt with context
            full_prompt = f"{prompt}, thick outlines, bold lines, children's coloring book style"
            
            # Step 3: Generate
            image = run_generation_pipeline(full_prompt)
            
            # Step 4: Store in state
            st.session_state.generated_image = image
            st.session_state.generated_image_original = image.copy()
            st.session_state.last_prompt = prompt
            st.session_state.step = 1
            st.session_state.generating = False
            
            # Step 5: Rerun to show result
            st.rerun()
        
        except Exception as e:
            st.error(f"😅 Oops! Let's try again. {str(e)[:50]}")
            st.session_state.generating = False
```

### 4. Error Handling UX (No Scary Red Boxes)

| Error Type | User Message | UI Element |
|-----------|--------------|-----------|
| Prompt too short | "💭 Tell me more! Use at least 3 words" | `st.info()` (blue) |
| Model loading slow | "🎨 Creating magic... (first time is slower!)" | `st.spinner()` |
| Generation OOM | "😴 That was too complex. Try something simpler!" | `st.warning()` (orange) |
| File upload wrong | "📁 That doesn't look like an image. Try again!" | `st.info()` (blue) |
| No color detected | "🎨 Make sure you used bold colors inside the lines!" | `st.info()` + debug image |

**Rule**: Never use `st.error()` for recoverable issues; use `st.warning()` or `st.info()` instead.

### 5. Mobile Responsiveness (Streamlit Default)

Streamlit auto-stacks columns on mobile. Ensure:
- Buttons are always full-width (`use_container_width=True`)
- Images scale proportionally (use `use_column_width=True`)
- Text is large enough (min 16px implicit)
- No nested columns deeper than 2 levels

```python
# ✅ GOOD: Mobile-friendly
col1, col2 = st.columns(2)
with col1:
    st.button("Button 1", use_container_width=True)

# ❌ AVOID: Nested columns get cramped
col1, col2 = st.columns(2)
with col1:
    col_inner1, col_inner2 = st.columns(2)
```

### 6. Image Display & Sizing

```python
# For large preview images (full width)
st.image(image, use_column_width=True, caption="Your coloring page")

# For side-by-side comparison (2 columns)
col1, col2 = st.columns(2)
with col1:
    st.image(original, use_column_width=True, caption="Original")
with col2:
    st.image(colored, use_column_width=True, caption="Your version")

# For GIF (should be centered and prominent)
col_space1, col_content, col_space2 = st.columns([0.2, 0.6, 0.2])
with col_content:
    st.image(gif_bytes, use_column_width=True)
```

### 7. Download Button UX

Always provide:
- **Large, visible button** with emoji
- **Clear filename** (includes step: `colorquest_page_001.png`)
- **Format clear** (PNG for stills, GIF for animations)
- **Post-download feedback** ("Your file is ready!")

```python
if st.session_state.generated_image:
    img_bytes = image_to_bytes(st.session_state.generated_image, format="PNG")
    st.download_button(
        label="📥 Download Coloring Page (PNG)",
        data=img_bytes,
        file_name=f"colorquest_page_{int(time.time())}.png",
        mime="image/png",
        use_container_width=True,
        key="download_page"
    )
    st.success("✅ Click the button above to save your page!")
```

---

## Input Validation & Constraints

### Prompt Input

```python
prompt = st.text_input(
    "What do you want to color?",
    placeholder="e.g., 'cute dragon flying'",
    max_chars=100
)

if len(prompt) < 3:
    st.warning("💭 Tell me more! At least 3 words please (e.g., 'blue dragon')")
    st.stop()

if len(prompt.split()) > 20:
    st.warning("📝 That's a lot! Try 2-5 words for clearer coloring pages.")
    prompt = " ".join(prompt.split()[:15])  # Auto-truncate
```

### File Upload Validation

```python
uploaded_file = st.file_uploader(
    "Upload your colored artwork",
    type=["jpg", "jpeg", "png"]
)

if uploaded_file:
    # Validate file size
    if uploaded_file.size > 10 * 1024 * 1024:  # 10MB
        st.error("📁 File too large. Try a file under 10MB.")
        st.stop()
    
    # Validate image dimensions
    try:
        image = Image.open(uploaded_file)
        if image.size[0] < 200 or image.size[1] < 200:
            st.warning("📱 Image very small. Larger images work better!")
        if image.size[0] > 4000 or image.size[1] > 4000:
            st.info("📏 Resizing large image for faster processing...")
            image = image.resize((2000, 2000), Image.LANCZOS)
    except Exception as e:
        st.error(f"📁 Not an image: {str(e)}")
        st.stop()
```

---

## Key Streamlit Config & Setup

```python
# config.toml (in ~/.streamlit/ or project root)
[client]
showErrorDetails = false
maxUploadSize = 10

[theme]
primaryColor = "#6366F1"       # Indigo (creative, playful)
backgroundColor = "#F8F9FA"    # Light gray (clean)
secondaryBackgroundColor = "#E0E7FF"  # Light indigo
textColor = "#1F2937"          # Dark gray (readable)
font = "sans serif"

[server]
maxMessageSize = 100

[logger]
level = "error"  # Hide debug logs from users
```

---

## Accessibility (WCAG 2.1)

- **Color contrast**: Ensure text is readable (dark text on light background ✅)
- **Button labels**: Always include text + emoji (not emoji-only)
- **Form fields**: Include helpful placeholder text & `help=` tooltips
- **Images**: Add captions describing what's shown
- **Keyboard navigation**: Streamlit buttons are keyboard-accessible by default

---

## Session State Lifecycle

```
App Load
  ↓
Initialize session_state
  ↓
LOOP: User interacts
  ├─ Sidebar: Enter prompt → Click "Generate"
  ├─ Callback: generate_coloring_page()
  ├─ Update: st.session_state.generated_image
  ├─ Rerun: st.rerun()
  ├─ Main area updates with result image
  │
  ├─ User uploads colored version
  ├─ Callback: process_colored_image()
  ├─ Update: st.session_state.upload_image
  ├─ Rerun: st.rerun()
  ├─ Main area shows side-by-side comparison
  │
  ├─ User selects animation style + clicks "Animate"
  ├─ Callback: create_animation()
  ├─ Update: st.session_state.animated_gif
  ├─ Rerun: st.rerun()
  ├─ Main area shows GIF preview + download button
  │
  ↓
(Cycle repeats for next creation)
```

---

## Performance & Responsiveness

| Action | Target Time | UX Strategy |
|--------|------------|------------|
| First app load | <2s | Show splash screen with logo while loading models |
| Prompt input validation | <100ms | Show inline validation (green ✓ / orange ⚠) |
| Generate image | <30s (draft) / <90s (final) | Show spinner + ETA "About 1 minute..." |
| Process upload | <5s | Show progress bar for image resize |
| Create animation | <10s | Show spinner "Creating animation frames..." |

```python
import time

with st.spinner("🎨 Creating your coloring page... This may take up to 90 seconds!"):
    start_time = time.time()
    image = generate_image_pipeline(prompt)
    elapsed = time.time() - start_time
    st.success(f"✅ Done in {elapsed:.1f}s!")
```

---

## Deployment Checklist

Before shipping to production:

- [ ] Test on mobile device (iPhone + Android)
- [ ] Test with slow internet (simulate 3G in DevTools)
- [ ] Verify all emojis render correctly (test on different browsers)
- [ ] Download buttons work (test file names, formats)
- [ ] Error messages are kid-friendly (no stack traces)
- [ ] Large images handled gracefully (auto-resize, no crash)
- [ ] Session state persists across rerun (test back button)
- [ ] Model caching works (check 2nd run is faster)

---

## UI/UX v2 Roadmap

1. **Mobile AR Integration**: Use phone camera to preview coloring page in AR before printing
2. **Social Sharing**: Generate shareable links, leaderboard of animations (top colored pages this week!)
3. **Templates Library**: Preset styles ("Dinosaur World", "Princess Castle", "Space Adventure") instead of text input
4. **Undo/Redo**: Save up to 3 generations in session for user to pick from
5. **Collaborative Coloring**: QR code to share a page with friends, vote on animation style

