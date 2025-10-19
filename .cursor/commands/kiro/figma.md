<meta>
description: Figma design integration for spec-driven development
argument-hint: [action] [parameters]
</meta>

# Figma Design Integration

Figma MCPを使用してデザインファイルから仕様を生成し、実装を加速します。

Tool policy: Use Cursor file tools + Figma MCP integration; no shell unless required for asset export.

## Usage

```bash
# Figmaデザインレビュー
/kiro/figma review [figma-url]

# コンポーネント詳細表示
/kiro/figma inspect [component-name]

# アセットエクスポート
/kiro/figma export [component-name]

# 変更差分確認
/kiro/figma diff [feature-name]

# デザイントークン抽出
/kiro/figma tokens

# ヘルプ
/kiro/figma --help
```

---

## Command: review

### Task: Review Figma Design File

Figmaデザインファイル全体をレビューし、概要を生成します。

**Usage**:
```bash
/kiro/figma review [figma-file-url]
/kiro/figma review https://www.figma.com/file/ABC123/Navigation-App
```

**Process**:

1. **Figma File Access**: Figma MCP経由でファイルにアクセス
2. **Structure Analysis**: ページ、フレーム、コンポーネント構造を解析
3. **Design System Extraction**: 色、フォント、コンポーネントを抽出
4. **Generate Report**: レビューレポートを生成

**Generate**: `.kiro/figma-review.md`

**Output Example**:
```markdown
# Figma Design Review: Navigation App

**File**: Navigation App
**URL**: https://www.figma.com/file/ABC123/Navigation-App
**Last Modified**: 2025-10-18 10:30:00
**Reviewed**: 2025-10-20 15:00:00

---

## Structure Overview

### Pages
1. 🎨 **Design System** (12 frames, 45 components)
2. 📱 **Flutter Screens** (8 screens)
3. ⌚ **watchOS Screens** (5 screens)
4. 🔗 **Prototypes** (2 flows)

---

## Design System

### Colors (Variables)
| Name | Value | Usage |
|------|-------|-------|
| Primary/Blue | #007AFF | Primary actions, active states |
| Secondary/LightBlue | #5AC8FA | Secondary UI elements |
| Success/Green | #34C759 | Success messages, confirmations |
| Warning/Orange | #FF9500 | Warnings, cautions |
| Error/Red | #FF3B30 | Errors, destructive actions |
| Background/Primary | #FFFFFF | Main backgrounds |
| Background/Secondary | #F2F2F7 | Secondary backgrounds |
| Text/Primary | #000000 | Primary text |
| Text/Secondary | #8E8E93 | Secondary text, hints |

### Typography (Text Styles)
| Style | Font | Size | Weight | Line Height |
|-------|------|------|--------|-------------|
| Heading/H1 | SF Pro Display | 34pt | Bold | 41pt |
| Heading/H2 | SF Pro Display | 28pt | Bold | 34pt |
| Heading/H3 | SF Pro Display | 22pt | Semibold | 28pt |
| Body/Regular | SF Pro Text | 17pt | Regular | 22pt |
| Body/Bold | SF Pro Text | 17pt | Bold | 22pt |
| Caption/Regular | SF Pro Text | 12pt | Regular | 16pt |

### Spacing (Variables)
| Name | Value | Usage |
|------|-------|-------|
| XXS | 2pt | Minimal spacing |
| XS | 4pt | Compact spacing |
| S | 8pt | Small spacing |
| M | 16pt | Medium spacing (most common) |
| L | 24pt | Large spacing |
| XL | 32pt | Extra large spacing |
| XXL | 48pt | Maximum spacing |

### Components
- **Buttons** (5 variants): Primary, Secondary, Tertiary, Destructive, Icon
- **Inputs** (4 variants): TextField, TextArea, SearchField, Picker
- **Cards** (3 variants): Elevated, Outlined, Filled
- **Navigation** (2 variants): TabBar, NavBar
- **Feedback** (3 variants): Alert, Toast, ProgressBar

---

## Flutter Screens

### 1. Home Screen
- **Size**: 375 x 812pt (iPhone 14)
- **Components**: NavBar, SearchField, MapPreview, QuickActions, TabBar
- **Prototype**: Linked to Navigation Screen

### 2. Navigation Screen
- **Size**: 375 x 812pt
- **Components**: MapView, RouteOverlay, BottomPanel, ActionButtons
- **Interactions**: 
  - Tap "Start" → Transition to Active Navigation
  - Swipe left → Show Alternate Routes
- **Prototype**: Full navigation flow

### 3. Settings Screen
- **Size**: 375 x 812pt
- **Components**: NavBar, SettingsList, Toggle, Slider
- **Prototype**: Basic interactions

---

## watchOS Screens

### 1. Watch Home
- **Size**: 184 x 224pt (Watch 45mm)
- **Components**: CompactMap, QuickStart, Complication
- **Prototype**: Digital Crown scroll

### 2. Compact Navigation
- **Size**: 184 x 224pt
- **Components**: RouteProgressRing, ETALabel, DistanceLabel
- **Prototype**: Linked from Watch Home

---

## Prototypes

### Flutter Flow
- **Screens**: 8 screens
- **Interactions**: 15 interactions
- **Key Flows**:
  1. Home → Search → Navigation → Active
  2. Navigation → Alternate Routes → Select → Active
  3. Active → Settings → Update → Resume

### watchOS Flow
- **Screens**: 5 screens
- **Interactions**: 8 interactions
- **Key Flows**:
  1. Watch Home → Quick Start → Navigation
  2. Navigation → Digital Crown (distance adjust)

---

## Recommendations

### For Requirements Phase
- [ ] Extract UI requirements from Flutter screens
- [ ] Extract interaction requirements from prototypes
- [ ] Define watchOS-specific requirements from watch screens
- [ ] Map sync requirements between platforms

### For Design Phase
- [ ] Use extracted design tokens for Flutter theme
- [ ] Use extracted design tokens for SwiftUI theme
- [ ] Map Figma components to Flutter widgets
- [ ] Map Figma components to SwiftUI views
- [ ] Document animation specs from prototypes

### For Implementation
- [ ] Export all assets (icons, images)
- [ ] Generate Flutter design system files
- [ ] Generate SwiftUI design system files
- [ ] Create component mapping documentation

---

## Next Steps

1. Initialize feature with Figma reference:
   \`\`\`bash
   /kiro/init feature "Navigation display with Figma design"
   \`\`\`

2. Generate requirements from Figma:
   \`\`\`bash
   /kiro/spec [feature] requirements --from-figma
   \`\`\`

3. Generate design with Figma integration:
   \`\`\`bash
   /kiro/spec [feature] design --with-figma
   \`\`\`

4. Export assets:
   \`\`\`bash
   /kiro/figma export NavigationScreen
   \`\`\`
```

---

## Command: inspect

### Task: Inspect Figma Component

特定のFigmaコンポーネントの詳細情報を表示します。

**Usage**:
```bash
/kiro/figma inspect [component-name]
/kiro/figma inspect NavigationScreen
/kiro/figma inspect Button/Primary
```

**Output**:
```markdown
# Figma Component: NavigationScreen

**Component ID**: 123:456
**Type**: Frame
**Page**: Flutter Screens
**Last Modified**: 2025-10-18 10:30:00

---

## Layout

- **Size**: 375 x 812pt
- **Auto Layout**: Vertical
- **Padding**: 0pt
- **Gap**: 0pt
- **Alignment**: Top Left
- **Constraints**: 
  - Horizontal: Left & Right
  - Vertical: Top & Bottom

---

## Style

### Fill
- Type: Solid
- Color: #FFFFFF (Background/Primary)
- Opacity: 100%

### Effects
- None

---

## Children (3 layers)

### 1. MapView (Frame)
- **Type**: Frame
- **Position**: (0, 0)
- **Size**: 375 x 692pt
- **Fill**: #FFFFFF
- **Children**: GoogleMap placeholder

### 2. RouteOverlay (Vector)
- **Type**: Vector
- **Position**: Absolute
- **Stroke**: #007AFF (Primary/Blue)
- **Stroke Width**: 4pt
- **Opacity**: 90%

### 3. BottomPanel (Frame)
- **Type**: Auto Layout Frame
- **Position**: (0, 692)
- **Size**: 375 x 120pt
- **Fill**: #FFFFFF
- **Corner Radius**: 20pt (top-left, top-right)
- **Shadow**: 
  - X: 0, Y: -2, Blur: 8, Spread: 0
  - Color: rgba(0,0,0,0.1)
- **Auto Layout**: Horizontal
- **Padding**: 16pt
- **Gap**: 12pt
- **Children**: 
  - StartButton (Component: Button/Primary)
  - SettingsButton (Component: Button/Icon)

---

## Assets Required

### Images
- None

### Icons
- navigation-start.svg (from StartButton)
- settings-icon.svg (from SettingsButton)

### Fonts
- SF Pro Display Bold 20pt (StartButton label)
- SF Pro Text Regular 14pt (BottomPanel labels)

---

## Implementation Hints

### Flutter
\`\`\`dart
// lib/features/navigation/presentation/pages/navigation_page.dart
class NavigationPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColors.background,  // #FFFFFF
      body: Stack(
        children: [
          // MapView (0, 0, 375x692)
          Positioned(
            top: 0,
            left: 0,
            right: 0,
            height: 692,
            child: MapView(),
          ),
          
          // RouteOverlay (absolute positioning)
          Positioned.fill(
            child: RouteOverlay(
              color: AppColors.primary,  // #007AFF
              strokeWidth: 4.0,
              opacity: 0.9,
            ),
          ),
          
          // BottomPanel (0, 692, 375x120)
          Positioned(
            bottom: 0,
            left: 0,
            right: 0,
            height: 120,
            child: Container(
              decoration: BoxDecoration(
                color: AppColors.background,
                borderRadius: BorderRadius.vertical(
                  top: Radius.circular(20),
                ),
                boxShadow: [
                  BoxShadow(
                    color: Colors.black.withOpacity(0.1),
                    offset: Offset(0, -2),
                    blurRadius: 8,
                  ),
                ],
              ),
              padding: EdgeInsets.all(16),
              child: Row(
                mainAxisAlignment: MainAxisAlignment.spaceBetween,
                children: [
                  PrimaryButton(
                    label: 'Start Navigation',
                    onPressed: () {},
                  ),
                  IconButton(
                    icon: Icon(Icons.settings),
                    onPressed: () {},
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
\`\`\`

### SwiftUI (if applicable)
\`\`\`swift
// Not directly applicable - this is Flutter-specific screen
// For watchOS, see CompactNavigationView
\`\`\`

---

## Export Assets

\`\`\`bash
/kiro/figma export NavigationScreen --output flutter_app/assets/
\`\`\`
```

---

## Command: export

### Task: Export Figma Assets

Figmaからアセット（アイコン、画像）をエクスポートします。

**Usage**:
```bash
/kiro/figma export [component-name]
/kiro/figma export NavigationScreen
/kiro/figma export Button/Primary --format svg
/kiro/figma export --all --output flutter_app/assets/
```

**Options**:
- `--format`: svg | png | pdf (default: svg for icons, png for images)
- `--output`: 出力先ディレクトリ
- `--scale`: 1x, 2x, 3x (PNG only)
- `--all`: すべてのアセットをエクスポート

**Process**:

1. **Identify Assets**: コンポーネント内のすべてのアセットを特定
2. **Export via Figma API**: Figma API経由でエクスポート
3. **Organize Files**: プラットフォーム別にファイルを整理
4. **Generate Manifest**: アセットマニフェストを生成

**Output Structure**:
```
flutter_app/assets/
├── icons/
│   ├── navigation-start.svg
│   ├── settings-icon.svg
│   └── ...
├── images/
│   ├── map-placeholder.png
│   ├── map-placeholder@2x.png
│   └── map-placeholder@3x.png
└── asset-manifest.json

watch_app/WatchApp/Assets.xcassets/
├── Icons/
│   ├── navigation-start.imageset/
│   │   ├── Contents.json
│   │   ├── navigation-start.svg
│   │   └── ...
│   └── ...
└── Images/
    └── ...
```

**Generate**: `asset-manifest.json`
```json
{
  "generated_at": "2025-10-20T15:00:00Z",
  "figma_file": "ABC123",
  "component": "NavigationScreen",
  "assets": {
    "icons": [
      {
        "name": "navigation-start",
        "format": "svg",
        "path": "assets/icons/navigation-start.svg",
        "figma_node_id": "123:789"
      }
    ],
    "images": [
      {
        "name": "map-placeholder",
        "format": "png",
        "scales": ["1x", "2x", "3x"],
        "paths": {
          "1x": "assets/images/map-placeholder.png",
          "2x": "assets/images/map-placeholder@2x.png",
          "3x": "assets/images/map-placeholder@3x.png"
        },
        "figma_node_id": "123:790"
      }
    ]
  }
}
```

**Auto-update pubspec.yaml** (Flutter):
```yaml
flutter:
  assets:
    - assets/icons/
    - assets/images/
```

---

## Command: diff

### Task: Figma Design Diff

Figmaデザインの変更を検出し、影響を受ける仕様を特定します。

**Usage**:
```bash
/kiro/figma diff [feature-name]
/kiro/figma diff navigation-display
```

**Prerequisites**:
- 機能仕様に Figma参照が含まれている
- 前回の Figma snapshot が存在する

**Process**:

1. **Load Previous Snapshot**: 前回のFigmaスナップショットを読み込む
2. **Fetch Current State**: 現在のFigmaデータを取得
3. **Compare**: 差分を計算
4. **Analyze Impact**: 仕様への影響を分析

**Output**:
```markdown
# Figma Design Diff: navigation-display

**Last Sync**: 2025-10-18 10:00:00
**Current Check**: 2025-10-20 15:00:00
**Figma File**: Navigation App

---

## Changes Detected

### Modified Components (3)

#### 1. BottomPanel
- **Change**: Layout updated
- **Details**:
  - Height: 120pt → 140pt (+20pt)
  - Padding: 16pt → 20pt (+4pt)
  - Gap: 12pt → 16pt (+4pt)
- **Impact**: 
  - ❗ **High**: Layout implementation needs update
  - Affected Files:
    - `flutter_app/lib/features/navigation/presentation/widgets/bottom_panel.dart`
  - Affected Specs:
    - `.kiro/specs/navigation-display/design.md` (BottomPanel component)
    - `.kiro/specs/navigation-display/tasks.md` (Task 2.4)

#### 2. RouteOverlay
- **Change**: Style updated
- **Details**:
  - Stroke Color: #007AFF → #0A84FF (lighter shade)
  - Opacity: 90% → 85% (-5%)
- **Impact**:
  - ⚠️ **Medium**: Visual update required
  - Affected Files:
    - `flutter_app/lib/features/navigation/presentation/widgets/route_overlay.dart`
  - Affected Specs:
    - `.kiro/specs/navigation-display/design.md` (RouteOverlay component)

#### 3. StartButton
- **Change**: Text style updated
- **Details**:
  - Font Size: 20pt → 18pt (-2pt)
  - Font Weight: Bold → Semibold
- **Impact**:
  - ℹ️ **Low**: Minor text style adjustment
  - Affected Files:
    - `flutter_app/lib/core/theme/app_text_styles.dart`

### New Components (1)

#### 4. AlternateRoutesPanel
- **Type**: Frame (Auto Layout)
- **Size**: 375 x 200pt
- **Position**: Below BottomPanel (slide-up animation)
- **Impact**:
  - ❗ **High**: New feature - requires specification
  - Action Required:
    - Add requirement for alternate routes display
    - Add design for AlternateRoutesPanel
    - Add implementation tasks

### Deleted Components (0)
- None

---

## Affected Specifications

### requirements.md
- **Status**: ⚠️ Update Recommended
- **Changes Needed**:
  - No new requirements needed for style changes
  - ❗ New requirement needed for AlternateRoutesPanel

### design.md
- **Status**: ❗ Update Required
- **Changes Needed**:
  - Update BottomPanel layout specs (height, padding, gap)
  - Update RouteOverlay color specs
  - Update StartButton text style
  - ❗ Add AlternateRoutesPanel component design

### tasks.md
- **Status**: ⚠️ Update Recommended
- **Changes Needed**:
  - Task 2.4 (BottomPanel): Update implementation specs
  - ❗ Add new task for AlternateRoutesPanel

---

## Recommended Actions

### 1. 🔴 Critical Updates (Do First)
```bash
# Add new requirement for AlternateRoutesPanel
/kiro/spec navigation-display requirements
# Manually add: "User can view and select alternate routes"

# Update design with Figma sync
/kiro/spec navigation-display design --sync-figma

# Update tasks
/kiro/spec navigation-display tasks --sync-figma
```

### 2. 🟡 Medium Updates
```bash
# Update RouteOverlay color in code
# flutter_app/lib/features/navigation/presentation/widgets/route_overlay.dart
# Change: Color(0xFF007AFF) → Color(0xFF0A84FF)
# Change: opacity: 0.9 → opacity: 0.85
```

### 3. 🟢 Minor Updates
```bash
# Update button text style
# flutter_app/lib/core/theme/app_text_styles.dart
# Update: buttonPrimary fontSize: 20 → 18, weight: Bold → Semibold
```

---

## Sync Summary

| Category | Count | Impact Level |
|----------|-------|--------------|
| Modified Components | 3 | High (1), Medium (1), Low (1) |
| New Components | 1 | High (1) |
| Deleted Components | 0 | - |
| **Total Changes** | **4** | **Critical Actions: 1** |

---

## Auto-Sync Option

自動的に仕様を更新する場合:
\`\`\`bash
/kiro/figma sync navigation-display --auto
\`\`\`

⚠️ 警告: 既存の手動編集が上書きされる可能性があります。
推奨: 手動でレビューしながら更新してください。
```

---

## Command: tokens

### Task: Extract Design Tokens

Figmaからデザイントークンを抽出し、コード生成します。

**Usage**:
```bash
/kiro/figma tokens
/kiro/figma tokens --output flutter_app/lib/core/theme/
/kiro/figma tokens --format dart
```

**Options**:
- `--output`: 出力先ディレクトリ
- `--format`: dart | swift | json (default: dart + swift)
- `--overwrite`: 既存ファイルを上書き

**Generate**:

**Flutter** (`app_colors.dart`, `app_text_styles.dart`, `app_spacing.dart`):
```dart
// Generated from Figma Variables - DO NOT EDIT MANUALLY
// Generated at: 2025-10-20 15:00:00
// Figma File: Navigation App

import 'package:flutter/material.dart';

class AppColors {
  AppColors._();
  
  // Primary Colors
  static const primary = Color(0xFF007AFF);
  static const secondary = Color(0xFF5AC8FA);
  
  // Status Colors
  static const success = Color(0xFF34C759);
  static const warning = Color(0xFFFF9500);
  static const error = Color(0xFFFF3B30);
  
  // Background Colors
  static const background = Color(0xFFFFFFFF);
  static const backgroundSecondary = Color(0xFFF2F2F7);
  
  // Text Colors
  static const textPrimary = Color(0xFF000000);
  static const textSecondary = Color(0xFF8E8E93);
}

class AppTextStyles {
  AppTextStyles._();
  
  static const h1 = TextStyle(
    fontFamily: 'SF Pro Display',
    fontSize: 34,
    fontWeight: FontWeight.w700,
    height: 41/34,
  );
  
  static const body = TextStyle(
    fontFamily: 'SF Pro Text',
    fontSize: 17,
    fontWeight: FontWeight.w400,
    height: 22/17,
  );
  
  static const caption = TextStyle(
    fontFamily: 'SF Pro Text',
    fontSize: 12,
    fontWeight: FontWeight.w400,
    height: 16/12,
  );
}

class AppSpacing {
  AppSpacing._();
  
  static const double xxs = 2.0;
  static const double xs = 4.0;
  static const double s = 8.0;
  static const double m = 16.0;
  static const double l = 24.0;
  static const double xl = 32.0;
  static const double xxl = 48.0;
}
```

**SwiftUI** (`AppColors.swift`, `AppTextStyles.swift`, `AppSpacing.swift`):
```swift
// Generated from Figma Variables - DO NOT EDIT MANUALLY
// Generated at: 2025-10-20 15:00:00
// Figma File: Navigation App

import SwiftUI

enum AppColors {
    // Primary Colors
    static let primary = Color(red: 0/255, green: 122/255, blue: 255/255)
    static let secondary = Color(red: 90/255, green: 200/255, blue: 250/255)
    
    // Status Colors
    static let success = Color(red: 52/255, green: 199/255, blue: 89/255)
    static let warning = Color(red: 255/255, green: 149/255, blue: 0/255)
    static let error = Color(red: 255/255, green: 59/255, blue: 48/255)
    
    // Background Colors
    static let background = Color.white
    static let backgroundSecondary = Color(red: 242/255, green: 242/255, blue: 247/255)
    
    // Text Colors
    static let textPrimary = Color.black
    static let textSecondary = Color(red: 142/255, green: 142/255, blue: 147/255)
}

enum AppTextStyles {
    static let h1 = Font.system(size: 34, weight: .bold, design: .default)
    static let body = Font.system(size: 17, weight: .regular, design: .default)
    static let caption = Font.system(size: 12, weight: .regular, design: .default)
}

enum AppSpacing {
    static let xxs: CGFloat = 2.0
    static let xs: CGFloat = 4.0
    static let s: CGFloat = 8.0
    static let m: CGFloat = 16.0
    static let l: CGFloat = 24.0
    static let xl: CGFloat = 32.0
    static let xxl: CGFloat = 48.0
}
```

**JSON** (`design-tokens.json`):
```json
{
  "generated_at": "2025-10-20T15:00:00Z",
  "figma_file": "Navigation App",
  "colors": {
    "primary": "#007AFF",
    "secondary": "#5AC8FA",
    "success": "#34C759",
    "warning": "#FF9500",
    "error": "#FF3B30",
    "background": "#FFFFFF",
    "backgroundSecondary": "#F2F2F7",
    "textPrimary": "#000000",
    "textSecondary": "#8E8E93"
  },
  "typography": {
    "h1": {
      "fontFamily": "SF Pro Display",
      "fontSize": 34,
      "fontWeight": 700,
      "lineHeight": 41
    },
    "body": {
      "fontFamily": "SF Pro Text",
      "fontSize": 17,
      "fontWeight": 400,
      "lineHeight": 22
    },
    "caption": {
      "fontFamily": "SF Pro Text",
      "fontSize": 12,
      "fontWeight": 400,
      "lineHeight": 16
    }
  },
  "spacing": {
    "xxs": 2,
    "xs": 4,
    "s": 8,
    "m": 16,
    "l": 24,
    "xl": 32,
    "xxl": 48
  }
}
```

---

## Integration with Spec Workflow

### Figma-Enhanced Requirements

```bash
/kiro/spec [feature] requirements --from-figma [frame-name]
```

自動的に:
1. Figmaフレームを解析
2. UIコンポーネントから要件を抽出
3. プロトタイプからインタラクション要件を抽出
4. EARS形式で requirements.md に追加

### Figma-Enhanced Design

```bash
/kiro/spec [feature] design --with-figma
```

自動的に:
1. Figmaコンポーネントをコードコンポーネントにマッピング
2. デザイントークンを設計に反映
3. アニメーション仕様を追加
4. アセットリストを生成

---

## Error Handling

### Figma Access Error
```
❌ エラー: Figmaファイルにアクセスできません

原因:
- Figma Access Tokenが無効
- ファイルへのアクセス権限がない
- ファイルが削除された

解決策:
1. .envファイルを確認: FIGMA_ACCESS_TOKEN
2. Figmaファイルのアクセス権限を確認
3. Figma URLが正しいか確認
```

### Component Not Found
```
⚠️ 警告: コンポーネント 'NavigationScreen' が見つかりません

確認事項:
- コンポーネント名が正しいか
- Figma内でコンポーネントが公開されているか
- 大文字・小文字を含む正確な名前を使用

利用可能なコンポーネント一覧:
/kiro/figma list
```

---

## Success Criteria

- [ ] Figma MCP経由でデザインファイルにアクセス可能
- [ ] デザイントークンがコードに自動変換される
- [ ] コンポーネントマッピングが明確
- [ ] アセットが適切にエクスポートされる
- [ ] デザイン変更の影響範囲が可視化される
- [ ] 仕様とFigmaの同期が維持される

ultrathink

