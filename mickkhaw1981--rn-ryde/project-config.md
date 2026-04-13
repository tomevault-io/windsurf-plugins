---
trigger: always_on
description: Use specification and guidelines as you build the app.
---

# Project instructions

Use specification and guidelines as you build the app.

Write the complete code for every step. Do not get lazy.

Your goal is to completely finish whatever I ask for.

## Tech Stack

- Frontend: React Native with Expo, Nativewind/Tailwind
- Backend: Neon Serverless Postgres
- Auth: Clerk
- Payments: Stripe

## Project structure

Users/mikekhaw/dev/react_native/rn_ryde
├── app
│ ├── (api)
│ │ ├── driver+api.ts
│ │ ├── hello+api.ts
│ │ └── user+api.ts
│ ├── (auth)
│ │ ├── \_layout.tsx
│ │ ├── hello-world.tsx
│ │ ├── sign-in.tsx
│ │ ├── sign-up.tsx
│ │ └── welcome.tsx
│ ├── (root)
│ │ ├── (tabs)
│ │ │ ├── \_layout.tsx
│ │ │ ├── chat.tsx
│ │ │ ├── home.tsx
│ │ │ ├── profile.tsx
│ │ │ └── rides.tsx
│ │ ├── \_layout.tsx
│ │ ├── book-ride.tsx
│ │ ├── confirm-ride.tsx
│ │ ├── find-ride.tsx
│ │ └── place-autocomplete.tsx
│ ├── \_layout.tsx
│ ├── global.css
│ └── index.tsx
├── assets
│ ├── fonts
│ │ ├── PlusJakartaSans-Bold.ttf
│ │ ├── PlusJakartaSans-BoldItalic.ttf
│ │ ├── PlusJakartaSans-ExtraBold.ttf
│ │ ├── PlusJakartaSans-ExtraBoldItalic.ttf
│ │ ├── PlusJakartaSans-ExtraLight.ttf
│ │ ├── PlusJakartaSans-ExtraLightItalic.ttf
│ │ ├── PlusJakartaSans-Italic.ttf
│ │ ├── PlusJakartaSans-Light.ttf
│ │ ├── PlusJakartaSans-LightItalic.ttf
│ │ ├── PlusJakartaSans-Medium.ttf
│ │ ├── PlusJakartaSans-MediumItalic.ttf
│ │ ├── PlusJakartaSans-Regular.ttf
│ │ ├── PlusJakartaSans-SemiBold.ttf
│ │ └── PlusJakartaSans-SemiBoldItalic.ttf
│ ├── icons
│ │ ├── arrow-down.png
│ │ ├── arrow-up.png
│ │ ├── back-arrow.png
│ │ ├── chat.png
│ │ ├── check.png
│ │ ├── close.png
│ │ ├── dollar.png
│ │ ├── email.png
│ │ ├── eyecross.png
│ │ ├── google_icon.svg
│ │ ├── google.png
│ │ ├── home.png
│ │ ├── list.png
│ │ ├── lock.png
│ │ ├── map.png
│ │ ├── marker.png
│ │ ├── out.png
│ │ ├── person.png
│ │ ├── pin.png
│ │ ├── point.png
│ │ ├── profile.png
│ │ ├── search.png
│ │ ├── selected-marker.png
│ │ ├── star.png
│ │ ├── target.png
│ │ └── to.png
│ └── images
│ ├── adaptive-icon.png
│ ├── check.png
│ ├── destination_icon.png
│ ├── direct_right_icon.png
│ ├── favicon.png
│ ├── get-started.png
│ ├── icon.png
│ ├── location_icon.png
│ ├── map_background.png
│ ├── message.png
│ ├── no-result.png
│ ├── onboarding1.png
│ ├── onboarding2.png
│ ├── onboarding3.png
│ ├── partial-react-logo.png
│ ├── react-logo.png
│ ├── react-logo@2x.png
│ ├── react-logo@3x.png
│ ├── signup-car.png
│ └── splash.png
├── components
│ ├── icons
│ │ └── index.tsx
│ ├── ApiTest.tsx
│ ├── CustomButton.tsx
│ ├── Divider.tsx
│ ├── DriverCard.tsx
│ ├── FormField.tsx
│ ├── GoogleTextInput.tsx
│ ├── Map.tsx
│ ├── Payment.tsx
│ ├── PlaceAutocomplete.tsx
│ ├── PrimaryButton.tsx
│ ├── ProgressIndicator.tsx
│ ├── RideCard.tsx
│ ├── RideLayout.tsx
│ └── SocialButton.tsx
├── constants
│ └── index.ts
├── docs
│ ├── 1_Onboarding
│ │ ├── 1.1_prompt_welcome_screens_UI.md
│ │ ├── 1.2_prompt_welcome_screens_swiper.md
│ │ ├── 1.2_task_welcome_swiper.md
│ │ └── repomix_react_native_swiper.md
│ ├── 2_Auth
│ │ ├── 2.1_prompt_signup_UI.md
│ │ ├── 2.2_task_signup_UI.md
│ │ ├── 2.3_prompt_signup_keyboard.md
│ │ └── 2.3_task_signup_keyboard.md
│ ├── 3_API_Routes
│ │ ├── 3.1_output_plan.md
│ │ ├── 3.1_prompt_create_plan_API.md
│ │ ├── 3.2_task_API.md
│ │ └── 3.3_prompt_insert_API_call.md
│ ├── 4_Maps
│ │ ├── 4.1a_plan_display_user_on_map.md
│ │ ├── 4.1b_code_display_user_on_map.md
│ │ ├── 4.1c_agent_display_user_on_map.md
│ │ ├── 4.2a_plan_display_driver_on_map.md
│ │ ├── 4.2b_plan_display_driver_on_map.md
│ │ ├── 4.2c_task_display_driver_on_map.md
│ │ ├── 4.3a_task_autocomplete.md
│ │ ├── repomix_rn_maps.md
│ │ ├── repomix_zustand.md
│ │ └── task_autocomplete.md
│ ├── 5_Stripe
│ │ ├── 5.1a_plan_stripe.md
│ │ ├── 5.1b_task_stripe.md
│ │ ├── payment-flow-diagram.mermaid
│ │ ├── payment-flow.md
│ │ ├── rn_maps_routes.md
│ │ ├── rn_modal.md
│ │ └── stripe-payment-integration.md
│ └── api-implementation.md
├── dummy_data
│ ├── mock_drivers.ts
│ └── mock_rides.json
├── ios
│ └── rnryde.xcodeproj
│ └── project.xcworkspace
│ ├── xcshareddata
│ │ └── IDEWorkspaceChecks.plist
│ └── contents.xcworkspacedata
├── lib
│ ├── cache.ts
│ ├── db.ts
│ ├── fetch.ts
│ ├── location.ts
│ ├── map.ts
│ └── utils.ts
├── scripts
│ ├── test-db-connection.js
│ ├── test-user-api.js
│ └── verify-db-schema.js
├── store
│ └── index.ts
├── types
│ ├── google-maps.d.ts
│ ├── images.d.ts
│ └── type.d.ts
├── .cursor-tasks.md
├── .cursorrules.md
├── .eslintrc.js
├── app.json
├── babel.config.js
├── how_to_do_great_work.md
├── metro.config.js
├── nativewind-env.d.ts
├── package.json
├── README.md
├── tailwind.config.js
└── tsconfig.json

## Type Rules

Follow these rules when working with types.

- When importing types, use `@/types`
- Name files like `example-types.ts`
- All types should go in `types`
- Make sure to export the types in `types/index.ts`
- Prefer interfaces over type aliases
- If referring to db types, use `@/db/schema` such as `SelectTodo` from `todos-schema.ts`

An example of a type:

`types/actions-types.ts`

```ts
export type ActionState<T> =
  | { isSuccess: true; message: string; data: T }
  | { isSuccess: false; message: string; data?: never };
```

And exporting it:

`types/index.ts`

```ts
export * from "./actions-types";
```

API Documentation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mickkhaw1981) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
