<!-- Inclusion Mode: Always -->

# Figma Integration with Kiro SDD

Figma MCPを使用したデザイン駆動開発の統合ガイド

## Overview

Figma MCPを活用して、デザインファイルから仕様書を生成し、実装まで一貫したワークフローを実現します。

---

## Figma MCP Setup

### Prerequisites
- Figma アカウント
- Figma Access Token
- Figma MCP Server のインストール

### Configuration

**`.figma-config.json`** をプロジェクトルートに配置:
```json
{
  "figma_access_token": "${FIGMA_ACCESS_TOKEN}",
  "figma_file_key": "YOUR_FIGMA_FILE_KEY",
  "target_platforms": {
    "flutter": {
      "enabled": true,
      "output_path": "flutter_app/lib/design_system/"
    },
    "swiftui": {
      "enabled": true,
      "output_path": "watch_app/WatchApp/DesignSystem/"
    }
  }
}
```

---

## Workflow Integration

### Phase 0: Figma Design Review (New Phase)

機能開発の前に、Figmaデザインをレビューし、仕様に落とし込みます。

```bash
/kiro/figma review [figma-file-url]
```

**生成されるもの**:
- デザイン概要（スクリーンショット付き）
- コンポーネント一覧
- デザイントークン（色、フォント、スペーシング）
- インタラクションフロー

**出力**: `.kiro/specs/[feature]/figma-review.md`

---

### Phase 1: Requirements (Figma-driven)

Figmaデザインから要件を抽出します。

```bash
/kiro/spec [feature-name] requirements --from-figma [figma-frame-id]
```

**Figmaから抽出される要件**:

#### UI Requirements
```markdown
### Requirement 1.1: Navigation Screen UI
**Target Platforms**: Flutter, watchOS
**Figma Reference**: [Frame: Navigation Screen](figma://...)

#### Acceptance Criteria (from Figma)
1. WHEN user opens navigation screen THEN system SHALL display map view
   - _Platform: Flutter_
   - _Figma Component: MapView_
   - _Design Specs: 375x812pt (iPhone), full screen_

2. WHEN route is active THEN system SHALL display route overlay
   - _Platform: Flutter_
   - _Figma Component: RouteOverlay_
   - _Design Specs: Blue (#007AFF), 4pt stroke width_

3. WHEN user views on watch THEN system SHALL display compact navigation
   - _Platform: watchOS_
   - _Figma Component: CompactNavigationView_
   - _Design Specs: 184x224pt (Watch 45mm)_
```

#### Interaction Requirements
```markdown
### Requirement 1.2: Navigation Interaction
**Figma Prototype**: [Prototype Link](figma://...)

#### Acceptance Criteria (from Prototype)
1. WHEN user taps "Start Navigation" THEN system SHALL transition to navigation mode
   - _Animation: Fade 300ms, Ease-in-out_
   - _Figma Interaction: Click → Navigate to Frame_

2. WHEN user swipes left THEN system SHALL show alternate routes
   - _Gesture: Swipe horizontal_
   - _Figma Interaction: Drag → Smart Animate_
```

---

### Phase 2: Design (Figma-enhanced)

Figmaデザインを参照しながら技術設計を生成します。

```bash
/kiro/spec [feature-name] design --with-figma
```

**Figmaデザインの技術設計への反映**:

#### Component Mapping

**design.md** にFigmaコンポーネントマッピングを追加:

```markdown
## Component Mapping: Figma → Code

### Flutter Components

| Figma Component | Flutter Widget | Responsibility |
|----------------|----------------|----------------|
| NavigationScreen | NavigationPage | Main navigation screen |
| MapView | GoogleMap widget | Interactive map display |
| RouteOverlay | CustomPaint | Route visualization |
| BottomPanel | BottomSheet | Controls and info |

**Implementation**:
\`\`\`dart
// lib/features/navigation/presentation/pages/navigation_page.dart
class NavigationPage extends StatelessWidget {
  // Based on Figma: NavigationScreen (375x812pt)
  // Component ID: 123:456
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          MapView(),           // Figma: MapView
          RouteOverlay(),      // Figma: RouteOverlay
          BottomPanel(),       // Figma: BottomPanel
        ],
      ),
    );
  }
}
\`\`\`

### watchOS Components

| Figma Component | SwiftUI View | Responsibility |
|----------------|--------------|----------------|
| CompactNavigationView | NavigationView | Main watch screen |
| RouteProgressRing | ProgressView | Route progress |
| ETALabel | Text | ETA display |

**Implementation**:
\`\`\`swift
// watch_app/WatchApp/Views/NavigationView.swift
struct NavigationView: View {
    // Based on Figma: CompactNavigationView (184x224pt)
    // Component ID: 789:012
    
    var body: some View {
        VStack {
            RouteProgressRing()  // Figma: RouteProgressRing
            ETALabel()          // Figma: ETALabel
        }
    }
}
\`\`\`
```

#### Design Tokens

```markdown
## Design Tokens (from Figma)

### Colors
\`\`\`dart
// Flutter: lib/core/theme/app_colors.dart
class AppColors {
  // From Figma Variables
  static const primary = Color(0xFF007AFF);      // Primary/Blue
  static const secondary = Color(0xFF5AC8FA);    // Secondary/Light Blue
  static const success = Color(0xFF34C759);      // Status/Success
  static const warning = Color(0xFFFF9500);      // Status/Warning
  static const error = Color(0xFFFF3B30);        // Status/Error
  
  // Backgrounds
  static const background = Color(0xFFFFFFFF);   // Background/Primary
  static const backgroundSecondary = Color(0xFFF2F2F7);  // Background/Secondary
}
\`\`\`

\`\`\`swift
// watchOS: watch_app/WatchApp/Theme/AppColors.swift
enum AppColors {
    // From Figma Variables
    static let primary = Color(red: 0/255, green: 122/255, blue: 255/255)
    static let secondary = Color(red: 90/255, green: 200/255, blue: 250/255)
    static let success = Color(red: 52/255, green: 199/255, blue: 89/255)
    static let warning = Color(red: 255/255, green: 149/255, blue: 0/255)
    static let error = Color(red: 255/255, green: 59/255, blue: 48/255)
}
\`\`\`

### Typography
\`\`\`dart
// Flutter: lib/core/theme/app_text_styles.dart
class AppTextStyles {
  // From Figma Text Styles
  static const h1 = TextStyle(
    fontFamily: 'SF Pro Display',
    fontSize: 34,
    fontWeight: FontWeight.bold,
    height: 41/34,  // Line height from Figma
  );
  
  static const body = TextStyle(
    fontFamily: 'SF Pro Text',
    fontSize: 17,
    fontWeight: FontWeight.normal,
    height: 22/17,
  );
  
  static const caption = TextStyle(
    fontFamily: 'SF Pro Text',
    fontSize: 12,
    fontWeight: FontWeight.normal,
    height: 16/12,
  );
}
\`\`\`

### Spacing
\`\`\`dart
// From Figma Auto Layout spacing
class AppSpacing {
  static const xxs = 2.0;   // Figma: XXS
  static const xs = 4.0;    // Figma: XS
  static const s = 8.0;     // Figma: S
  static const m = 16.0;    // Figma: M
  static const l = 24.0;    // Figma: L
  static const xl = 32.0;   // Figma: XL
  static const xxl = 48.0;  // Figma: XXL
}
\`\`\`
```

#### Animation Specifications

```markdown
## Animation Specifications (from Figma Prototype)

### Screen Transitions
| Transition | From | To | Duration | Easing |
|-----------|------|-----|----------|--------|
| Navigation Start | Home | Navigation | 300ms | Ease-in-out |
| Route Change | Route A | Route B | 200ms | Ease-out |
| Bottom Sheet | Collapsed | Expanded | 250ms | Ease-in-out |

**Flutter Implementation**:
\`\`\`dart
class AppAnimations {
  static const navigationTransition = Duration(milliseconds: 300);
  static const routeChange = Duration(milliseconds: 200);
  static const bottomSheetExpand = Duration(milliseconds: 250);
  
  static const easeInOut = Curves.easeInOut;
  static const easeOut = Curves.easeOut;
}
\`\`\`

**SwiftUI Implementation**:
\`\`\`swift
enum AppAnimations {
    static let navigationTransition = Animation.easeInOut(duration: 0.3)
    static let routeChange = Animation.easeOut(duration: 0.2)
    static let bottomSheetExpand = Animation.easeInOut(duration: 0.25)
}
\`\`\`
```

---

### Phase 3: Implementation (Figma-guided)

実装時にFigmaデザインを参照できるようにします。

#### Figma Inspector Integration

```bash
# 実装中にFigmaデザインを確認
/kiro/figma inspect [component-name]
```

**出力例**:
```markdown
## Figma Component: NavigationScreen

### Layout
- Width: 375pt (iPhone 14)
- Height: 812pt
- Auto Layout: Vertical
- Padding: 0pt
- Gap: 0pt

### Children
1. MapView (Frame)
   - Position: (0, 0)
   - Size: 375 x 692
   - Fill: #FFFFFF
   
2. BottomPanel (Frame)
   - Position: (0, 692)
   - Size: 375 x 120
   - Fill: #FFFFFF
   - Corner Radius: 20pt (top corners)
   - Shadow: 0px -2px 8px rgba(0,0,0,0.1)

### Assets
- Icons: 12 icons needed
  - navigation-start.svg
  - route-alt.svg
  - settings.svg
  ...

### Export for Development
\`\`\`bash
# Export all assets
/kiro/figma export assets --component NavigationScreen --output flutter_app/assets/
\`\`\`
```

---

## Figma-Driven Task Generation

タスク生成時、Figmaコンポーネント単位でタスクを分割します。

**tasks.md** (Figma-enhanced):

```markdown
# Implementation Plan

## Design System Setup (Figma-derived)

- [ ] 1. Design System基盤の構築
- [ ] 1.1 デザイントークンの実装
  - Figmaから色・フォント・スペーシングを抽出
  - Flutter: `app_colors.dart`, `app_text_styles.dart`, `app_spacing.dart`
  - watchOS: `AppColors.swift`, `AppTextStyles.swift`, `AppSpacing.swift`
  - _Figma Reference: Design Tokens page_
  - _Requirements: All (デザイン一貫性)_

- [ ] 1.2 共通コンポーネントの実装
  - Figmaの基本コンポーネントをコード化
  - Button, TextField, Card 等
  - _Figma Components: Button/*, Input/*, Card/*_
  - _Requirements: All (UI consistency)_

## Flutter Implementation (Figma-guided)

- [ ] 2. Navigation Screen の実装
- [ ] 2.1 NavigationPage 画面構築
  - Figma: NavigationScreen (375x812pt) を再現
  - MapView, RouteOverlay, BottomPanel の統合
  - _Figma Component: NavigationScreen (ID: 123:456)_
  - _Requirements: 1.1, 1.2_
  - _Platform: Flutter_

- [ ] 2.2 MapView ウィジェットの実装
  - Google Maps統合
  - Figmaのデザインスペックに従う
  - _Figma Component: MapView (ID: 123:457)_
  - _Requirements: 1.1_
  - _Platform: Flutter_

- [ ] 2.3 RouteOverlay カスタムペイント実装
  - CustomPaint で経路描画
  - 色: #007AFF, 太さ: 4pt (Figma準拠)
  - _Figma Component: RouteOverlay (ID: 123:458)_
  - _Requirements: 1.2_
  - _Platform: Flutter_

- [ ] 2.4 BottomPanel 実装
  - Figmaのコーナー半径・影を再現
  - アニメーション: 250ms ease-in-out
  - _Figma Component: BottomPanel (ID: 123:459)_
  - _Figma Animation: Collapse/Expand_
  - _Requirements: 1.3_
  - _Platform: Flutter_

## watchOS Implementation (Figma-guided)

- [ ] 3. Compact Navigation View の実装
- [ ] 3.1 NavigationView SwiftUI構築
  - Figma: CompactNavigationView (184x224pt) を再現
  - RouteProgressRing, ETALabel の統合
  - _Figma Component: CompactNavigationView (ID: 789:012)_
  - _Requirements: 2.1_
  - _Platform: watchOS_

- [ ] 3.2 RouteProgressRing 実装
  - ProgressView カスタマイズ
  - 色: #007AFF (Figma準拠)
  - _Figma Component: RouteProgressRing (ID: 789:013)_
  - _Requirements: 2.1_
  - _Platform: watchOS_

## Figma Asset Export & Integration

- [ ] 4. アセット統合
- [ ] 4.1 Figmaから全アセットをエクスポート
  - アイコン: SVG形式
  - 画像: @1x, @2x, @3x
  - _Figma Assets: All icons and images_

- [ ] 4.2 Flutter アセット統合
  - `pubspec.yaml` に追加
  - `assets/icons/`, `assets/images/` に配置
  - _Platform: Flutter_

- [ ] 4.3 watchOS アセット統合
  - Xcode Asset Catalog に追加
  - SF Symbols使用検討
  - _Platform: watchOS_
```

---

## Figma Sync Workflow

### デザイン変更時の同期

Figmaデザインが更新された場合のワークフロー:

```bash
# 1. Figmaの変更を検出
/kiro/figma diff [feature-name]

# 出力例:
# 🔄 Figma Changes Detected
# 
# Changed Components:
# - NavigationScreen: Layout updated (Bottom panel height: 120pt → 140pt)
# - RouteOverlay: Color updated (#007AFF → #0A84FF)
# - BottomPanel: Corner radius updated (20pt → 24pt)
# 
# New Components:
# - AlternateRoutesPanel
# 
# Deleted Components:
# - (none)
# 
# Affected Specs:
# - .kiro/specs/navigation-display/requirements.md (1 requirement)
# - .kiro/specs/navigation-display/design.md (3 components)
# - .kiro/specs/navigation-display/tasks.md (2 tasks)

# 2. 仕様を更新
/kiro/spec navigation-display design --sync-figma

# 3. タスクを更新
/kiro/spec navigation-display tasks --sync-figma

# 4. 影響を確認
/kiro/status navigation-display --figma-diff
```

---

## Best Practices

### 1. Figmaファイル構造

**推奨構造**:
```
Figma File: Navigation App
├── 📄 Cover (プロジェクト概要)
├── 🎨 Design System
│   ├── Colors (変数として定義)
│   ├── Typography (テキストスタイルとして定義)
│   ├── Spacing (変数として定義)
│   └── Components (再利用可能コンポーネント)
├── 📱 Flutter Screens
│   ├── Home Screen
│   ├── Navigation Screen
│   └── Settings Screen
├── ⌚ watchOS Screens
│   ├── Watch Home
│   ├── Compact Navigation
│   └── Complications
└── 🔗 Prototypes
    ├── Flutter Flow
    └── watchOS Flow
```

### 2. コンポーネント命名規則

Figmaとコードで一貫した命名:

| Figma | Flutter | SwiftUI |
|-------|---------|---------|
| NavigationScreen | NavigationPage | NavigationView |
| Button/Primary | PrimaryButton | PrimaryButton |
| Input/TextField | AppTextField | AppTextField |
| Card/Elevated | ElevatedCard | ElevatedCard |

### 3. Figma Variables の活用

デザイントークンはFigma Variablesとして定義:
- Colors: `Primary/Blue`, `Secondary/LightBlue`
- Numbers: `Spacing/S`, `Spacing/M`, `Spacing/L`
- Strings: `FontFamily/Display`, `FontFamily/Text`

### 4. Auto Layout の活用

FlutterのColumn/RowやSwiftUIのVStack/HStackに対応させる:
- Figma Auto Layout → Flutter Column/Row
- Figma Padding → Flutter EdgeInsets
- Figma Gap → Flutter SizedBox / Spacer

---

## Figma MCP Commands Reference

### 統合コマンド

```bash
# Figmaデザインレビュー
/kiro/figma review [figma-url]

# Figmaから要件生成
/kiro/spec [feature] requirements --from-figma [frame-id]

# Figma参照で設計生成
/kiro/spec [feature] design --with-figma

# Figmaコンポーネント詳細表示
/kiro/figma inspect [component-name]

# Figmaアセットエクスポート
/kiro/figma export assets --component [name] --output [path]

# Figma変更差分確認
/kiro/figma diff [feature-name]

# Figma同期
/kiro/spec [feature] design --sync-figma
/kiro/spec [feature] tasks --sync-figma
```

---

## Security & Access Control

### Figma Access Token 管理

**`.env`** ファイル（Gitignore済み）:
```bash
FIGMA_ACCESS_TOKEN=your_figma_access_token_here
```

**`.gitignore`** に追加:
```
.env
.figma-config.json
```

### チーム共有

チーム用の設定テンプレート:
```bash
# .figma-config.template.json (Gitにコミット)
{
  "figma_access_token": "${FIGMA_ACCESS_TOKEN}",
  "figma_file_key": "YOUR_SHARED_FIGMA_FILE_KEY",
  "target_platforms": {
    "flutter": {
      "enabled": true,
      "output_path": "flutter_app/lib/design_system/"
    },
    "swiftui": {
      "enabled": true,
      "output_path": "watch_app/WatchApp/DesignSystem/"
    }
  }
}
```

チームメンバーは:
```bash
cp .figma-config.template.json .figma-config.json
# 自分のFIGMA_ACCESS_TOKENを.envに設定
```

---

## Troubleshooting

### Q: Figma MCPが接続できない
**A**: アクセストークンを確認:
```bash
echo $FIGMA_ACCESS_TOKEN
```

### Q: コンポーネントが見つからない
**A**: Figma内でコンポーネントが公開されているか確認。Component IDではなくComponent Nameで参照。

### Q: デザイントークンが反映されない
**A**: Figma Variablesとして定義されているか確認。ローカルスタイルではなくVariablesを使用。

---

## Summary

Figma MCPを統合することで:
- ✅ デザインから仕様への自動変換
- ✅ Flutter/SwiftUI実装の精度向上
- ✅ デザイン変更の影響範囲可視化
- ✅ デザイントークンの一元管理
- ✅ チーム全体のデザイン一貫性

Kiro SDD v2のワークフローに完全に統合可能です。

