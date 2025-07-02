# Dev Book Club: A Philosophy of Software Design
**Prepared by Guido – Chainels Frontend Engineer**

---
## 1. Warm-up (10 min)

### Q: What's one idea or chapter that stood out to you?

**A:** The framing of complexity around three symptoms — **Change Amplification**, **Cognitive Load**, and **Unknown Unknowns**. These apply directly to our React code, especially when dealing with legacy integrations or stateful flows.

**Change Amplification**: "A seemingly simple change requires code modifications in many different places." In our `BookingModal.tsx`, changing booking logic touches 5+ files across React, jQuery, and PHP.

**Cognitive Load**: "How much a developer needs to know in order to complete a task." Our multi-step forms require understanding `useMultiStepForm`, `DynamicForm`, Yup schemas, jQuery events, and slot availability logic simultaneously.

**Unknown Unknowns**: "It is not obvious which pieces of code must be modified to complete a task." Our legacy event system (`chn:legacy:booking:modal:open`) has no type contracts or clear ordering - if you forget to dispatch an event or the DOM hasn't rendered, it silently fails.

### Q: How would you describe this book in one sentence?

**A:** When your code feels messy but you can't quite put your finger on why. You can use the three symptoms to watch for (change amplification, cognitive load, unknown unknowns) and provide the tools and techniques to prevent and reduce complexity.

---

## 2. Main Discussion (60 min)

## Complexity is the Enemy
Complexity makes code harder to understand and modify.

### In our codebase, that's `BookingModal.tsx`, which coordinates:
- **Legacy JS bridge**
- **React form flow**
- **Event-based slot selection**
- **Dynamic schema building**

Changing one part often touches 3–5 files. That's **Change Amplification**.

### The Core Principle from Chapter 3:
> "If you program tactically, you will finish your first projects 10–20% faster, but over time your development speed will slow as complexity accumulates. It won't be long before you're programming at least 10–20% slower. You will quickly give back all of the time you saved at the beginning, and for the rest of system's lifetime you will be developing more slowly than if you had taken the strategic approach."

---

## Cognitive Load
Devs need to juggle: multi-step forms, dynamic validations, legacy events, translations, etc.

A new dev touching booking has to learn: `useMultiStepForm`, `DynamicForm`, Yup schemas, jQuery events, and slot availability logic.

Each part is manageable — but the aggregate is where the cost lies.

---

## Unknown Unknowns
Legacy systems wired by events (`chn:legacy:booking:modal:open`) have no type contracts or clear ordering.

If you forget to dispatch an event or the DOM hasn't rendered, it silently fails.

React can't help you here — that's where unknown unknowns bite.

---

## Tactical vs. Strategic Programming

### Tactical: "Let's just wire it up and move on."
### Strategic: "Let's build something we won't regret in 6 months."

### Where we've been tactical:
- Using jQuery glue code to patch legacy booking UI into React
- Manually building validation schemas outside `DynamicForm`
- Creating slot selection components that rely on external state refs

### What strategic could look like:
- A `useBookingFlow()` hook that owns the full lifecycle (slot → form → confirm)
- A `BookingForm` component that owns its schema, layout, and validation logic internally
- Migrating legacy triggers into declarative context or effects — no `$(document).on(...)`

### Real-World Examples from the Book:

### Why This Matters for Chainels:
As a growing company, we face the same pressures as early-stage startups:
- Pressure to ship features quickly
- Limited engineering resources
- Need to demonstrate value to stakeholders

However, the payoff for good design comes quickly. A tactical approach won't even speed up our first product release, and once a codebase turns to spaghetti, it's nearly impossible to fix.

---

## Deep vs. Shallow Modules

### Deep: small surface, hides complexity
→ `DynamicForm`, `useMultiStepForm`, (future) `useBookingFlow()`

### Shallow: large surface, little benefit
→ `capitalizeFirstLetter()`, or hooks that require manual `init()` steps

### From Chapter 4:
> "One of the most important techniques for managing software complexity is to design systems so that developers only need to face a small fraction of the overall complexity at any given time."

### Red Flag: Shallow Module
> "A shallow module is one whose interface is complicated relative to the functionality it provides. Shallow modules don't help much in the battle against complexity, because the benefit they provide (not having to learn about how they work internally) is negated by the cost of learning and using their interfaces. Small modules tend to be shallow."

---

## Temporal Coupling
**Legacy example:**
```javascript
$(document).on('chn:legacy:booking:modal:open', handleModalOpen);
```

React is declarative. This isn't. (declarative approach is when you describe the final state of the desired UI).

Hidden dependencies between event order and DOM structure are fragile.

---

## 3. Wrap-up (15–20 min)

### Q: What's one idea you want to apply or think more about?
**A:** Consolidate booking logic into a deep module (`useBookingFlow`). Reduce friction points by collapsing steps into one managed context and replace DOM events with state-driven flows.

### Q: Anything in the book you strongly disagreed with?
**A:** Not from this book. But Ousterhout challenges the Clean Code mindset that "comments are a failure."
He argues — and I agree — that comments are essential for explaining non-obvious logic or decisions, especially in large or legacy systems:

> "Well-written comments are not failures. They increase the value of code."

### Q: Do we want to do a follow-up or experiment with something from the book?
**A:** Yes. Let's prototype a version of the booking flow built strategically.

---

## Codebase Examples (Referenced in Discussion)

### Change Amplification

```typescript
const initialValues = React.useMemo(() => {
  return query.data?.questions.reduce((cur, question) => {
    cur[question.id] = getQuestionInitialValue(question, null);
    return cur;
  }, {} as SurveyFormAnswers);
}, [query.data?.questions]);
```

**The Problem**: To change question handling, you must touch:
1. `BookingModal.tsx` (React component)
2. `getQuestionInitialValue` helper
3. `SurveyFormAnswers` type definition
4. Query data structure
5. Form validation logic

### Cognitive Load

```typescript
const steps = React.useMemo(() => {
  const hasQuestions = query.data?.questions.length > 0;
  const questionsStep = hasQuestions && {
    component: <DynamicForm questions={query.data?.questions ?? []} />,
    validationSchema: Yup.object(
      query.data?.questions.reduce(...) // complex and verbose
    )
  };
});
```

**Cognitive Load**: Developer must understand:
- Multi-step form logic
- Dynamic form generation
- Yup validation schemas
- Translation keys
- Event handling
- Theme system

### Deep Module

```typescript
function DynamicForm({ questions }: DynamicFormProps) {
  const questionsComponents = React.useMemo(() => {
    return questions.map((question) => (
      <DynamicFormQuestion key={question.id} question={question} />
    ));
  }, [questions]);
  return <FormGroup>{questionsComponents}</FormGroup>;
}
```

**Benefits**: 
- Simple interface (just `questions` prop)
- Hides complex question rendering logic
- Handles validation, accessibility, styling internally

### Temporal Coupling

```javascript
$(document).on('chn:legacy:booking:modal:open', handleModalOpen);
function handleModalOpen() {
  const form = document.getElementById('booking-form');
  if (form) new BookingForm(form);
}
```

**Problems**:
- Hidden dependencies between event order and DOM structure
- No type contracts
- Silent failures if DOM isn't ready
- React can't help with this

---

## Additional Code Examples: Our Codebase Through the Lens of "A Philosophy of Software Design"

### 1. Change Amplification Examples

#### BookingModal.tsx - The Legacy Bridge Problem
```typescript
// Current: Change amplification in action
const handleSlotSelectionRef = React.useCallback(
  (slotSelection: HTMLDivElement | null) => {
    if (slotSelection && show) {
      document.dispatchEvent(
        new CustomEvent('chn:legacy:booking:modal:open', {
          detail: { bookableId },
        }),
      );
    }
  },
  [show, bookableId],
);

// To change booking flow, you must touch:
// 1. BookingModal.tsx (React component)
// 2. legacyBookingForm.js (jQuery code)
// 3. BookingPage.tsx (event listener)
// 4. PHP backend (event handling)
// 5. HTML templates (event dispatching)
```

**The Problem**: A simple change to booking logic requires modifications in 5+ files across different technologies.

#### TargetingDropdownContext - The Monolithic Context
```typescript
// Current: 15+ concerns in one context
const {
  include,
  exclude,
  mode,
  category,
  search,
  targets,
  navigation,
  display,
  counters,
  // ... 6+ more properties
} = useTargetingDropdown();

// To change targeting logic, you must understand:
// 1. Selection state management
// 2. Category navigation
// 3. Remote data fetching
// 4. Display logic
// 5. Counter calculations
// 6. Search functionality
// 7. Mode switching
// 8. Navigation state
```

**The Problem**: Every targeting change requires understanding 8+ different concerns.

### 2. Cognitive Load Examples

#### Multi-Step Form Complexity
```typescript
// Current: High cognitive load
const steps = React.useMemo(() => {
  const hasQuestions = query.data?.questions && query.data?.questions.length > 0;

  const bookingStep = [
    {
      title: t('BookingDictionary:booking_select_slots'),
      component: (
        <LegacySlotSelectionStep
          handleSlotSelectionRef={handleSlotSelectionRef}
          communityId={communityId ?? ''}
          bookableId={bookableId ?? ''}
          theme={theme}
        />
      ),
      validationSchema: Yup.object({}),
    },
  ];

  const questionsStep = hasQuestions && {
    title: t('TurnoverReportingDictionary:report_title_questions'),
    component: <DynamicForm questions={query.data?.questions ?? []} />,
    validationSchema: Yup.object(
      query.data?.questions.reduce(
        (acc, question) => {
          // Complex schema building logic
          const questionSchema = getValidationSchema(question, t);
          const questionFields = questionSchema.fields;
          Object.assign(acc, questionFields);
          return acc;
        },
        {} as Record<string, Yup.AnySchema>,
      ) || {},
    ),
  };

  return [...bookingStep, ...(questionsStep ? [questionsStep] : [])];
}, [t, query.data?.questions, handleSlotSelectionRef, communityId, bookableId, theme]);
```

**Cognitive Load**: Developer must understand:
- Multi-step form logic
- Legacy slot selection
- Dynamic form generation
- Yup validation schemas
- Translation keys
- Event handling
- Theme system

#### Table State Management
```typescript
// Current: Complex state management
function tableStateReducer(
  state: DynamicTableState,
  action: TableStateAction,
): DynamicTableState {
  switch (action.type) {
    case 'set_filters': {
      // Complex filter merging logic
      const filters =
        action.filters.length > 0
          ? [
              ...state.filters.filter(
                (stateFilter) =>
                  !action.filters.some(
                    (actionFilter) =>
                      actionFilter.column === stateFilter.column,
                  ),
              ),
              ...action.filters,
            ].filter((filter) => {
              if (Array.isArray(filter.value)) {
                return filter.value.length > 0;
              }
              return filter.value.value !== '';
            })
          : [];

      return {
        ...state,
        filters,
        selection: {
          ...state.selection,
          selectedRows: [],
          allRowsSelected: false,
          selectedRowCount: 0,
        },
        pagination: { pageSize: state.pagination.pageSize, pageIndex: 0 },
      };
    }
    // ... 8+ more cases
  }
}
```

**Cognitive Load**: Developer must understand:
- Filter state management
- Selection state logic
- Pagination reset rules
- Complex filter merging
- State synchronization

### 3. Deep vs Shallow Modules Examples

#### Deep Module (Good): DynamicForm
```typescript
// Deep: Simple interface, complex implementation
function DynamicForm({ questions }: DynamicFormProps) {
  const questionsComponents = React.useMemo(() => {
    return questions.map((question) => (
      <DynamicFormQuestion key={question.id} question={question} />
    ));
  }, [questions]);
  
  return <FormGroup>{questionsComponents}</FormGroup>;
}

// Usage: Simple and focused
<DynamicForm questions={surveyQuestions} />
```

**Benefits**: 
- Simple interface (just `questions` prop)
- Hides complex question rendering logic
- Handles validation, accessibility, styling internally

#### Shallow Module (Bad): One-liner Utilities
```typescript
// Shallow: Complex interface, simple implementation
function capitalizeFirstLetter(str: string): string {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

// Usage: Simple but adds API surface without hiding complexity
const title = capitalizeFirstLetter(userName);
```

**Problems**:
- Adds API surface without meaningful abstraction
- Developer must learn function name and signature
- No complexity hidden

#### Shallow Module (Bad): Manual Initialization Hooks
```typescript
// Shallow: Requires manual initialization
function useSomething() {
  const [isInitialized, setIsInitialized] = useState(false);
  
  const init = useCallback(() => {
    // Complex initialization logic
    setIsInitialized(true);
  }, []);
  
  return { isInitialized, init };
}

// Usage: Temporal coupling
const { isInitialized, init } = useSomething();
useEffect(() => {
  if (!isInitialized) {
    init(); // Developer must remember to call this
  }
}, [isInitialized, init]);
```

**Problems**:
- Temporal coupling (must call `init()` first)
- Developer must remember initialization order
- No complexity hidden, just moved around

### 4. Information Hiding Examples

#### Good Information Hiding: useLocalStorage
```typescript
// Good: Hides localStorage complexity
function useLocalStorage<T>({ storageKey }: { storageKey: string }) {
  let localStorageData: string | null;
  try {
    localStorageData = localStorage.getItem(storageKey);
  } catch (error) {
    localStorageData = null; // Handles errors internally
  }

  const parsedStorageData: T | null = localStorageData
    ? JSON.parse(localStorageData)
    : null;

  function setLocalStorage(newFormData: T) {
    if (!storageKey || !newFormData) return;
    localStorage.setItem(storageKey, JSON.stringify(newFormData));
  }

  return {
    parsedStorageData,
    setLocalStorage,
    removeLocalStorage,
  };
}

// Usage: Simple interface
const { parsedStorageData, setLocalStorage } = useLocalStorage({
  storageKey: 'form-data',
});
```

**Benefits**: Hides error handling, JSON parsing, and localStorage API complexity.

#### Bad Information Hiding: Leaking Implementation Details
```typescript
// Bad: Leaks Axios configuration
function useApi() {
  const api = axios.create({
    baseURL: process.env.API_URL,
    headers: {
      Authorization: `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    timeout: 5000,
  });
  
  return { api }; // Leaks Axios instance
}

// Usage: Consumer must understand Axios
const { api } = useApi();
const response = await api.get('/users'); // Consumer knows about Axios
```

**Problems**: Consumer must understand Axios API, breaking information hiding.

### 5. Temporal Coupling Examples

#### Legacy Event System
```typescript
// Current: Temporal coupling with events
$(document).on('chn:legacy:booking:modal:open', handleModalOpen);

function handleModalOpen() {
  const form = document.getElementById('booking-form');
  if (form) {
    new BookingForm(form); // Depends on DOM being ready
  }
}

// Usage: Hidden dependencies
document.dispatchEvent(new CustomEvent('chn:legacy:booking:modal:open'));
// Must happen after DOM is rendered, but no type safety or clear ordering
```

**Problems**:
- Hidden dependencies between event order and DOM structure
- No type contracts
- Silent failures if DOM isn't ready
- React can't help with this

#### App Initialization Hell
```typescript
// Current: Complex initialization order
React.useEffect(() => {
  setInitialized(); // Must happen first
}, [setInitialized]);

React.useEffect(() => {
  appStateDispatch({
    type: 'sidebar_nav',
    sidebar_nav: {
      /* complex nav structure */
    },
  });
}, [appStateDispatch]); // Must happen after initialization

React.useEffect(() => {
  if (query.data) {
    appStateDispatch({
      type: 'environments',
      environments: query.data,
    });
  }
}, [query.data, appStateDispatch]); // Must happen after nav setup
```

**Problems**:
- Complex initialization order
- Unknown unknowns about what breaks if order changes
- Difficult to debug initialization issues

### 6. Strategic vs Tactical Programming Examples

#### Tactical: Quick jQuery Integration
```typescript
// Tactical: Quick integration, long-term pain
const handleSlotSelectionRef = React.useCallback(
  (slotSelection: HTMLDivElement | null) => {
    if (slotSelection && show) {
      document.dispatchEvent(
        new CustomEvent('chn:legacy:booking:modal:open', {
          detail: { bookableId },
        }),
      );
    }
  },
  [show, bookableId],
);
```

**Tactical Benefits**: Got booking working quickly
**Tactical Costs**: Now every booking change requires touching React + jQuery + PHP

#### Strategic: Deep Module Design
```typescript
// Strategic: Investment in clean architecture
function useBookingFlow(bookableId: string) {
  const [step, setStep] = useState<'slots' | 'form' | 'confirm'>('slots');
  const [selectedSlots, setSelectedSlots] = useState<Slot[]>([]);
  const [formData, setFormData] = useState<BookingFormData>({});
  
  const handleSlotSelect = useCallback((slot: Slot) => {
    setSelectedSlots((prev) => [...prev, slot]);
  }, []);
  
  const handleFormSubmit = useCallback((data: BookingFormData) => {
    setFormData(data);
    setStep('confirm');
  }, []);
  
  const handleConfirm = useCallback(async () => {
    // Clean, declarative booking flow
    await createBooking({ slots: selectedSlots, formData });
  }, [selectedSlots, formData]);
  
  return {
    step,
    selectedSlots,
    formData,
    handleSlotSelect,
    handleFormSubmit,
    handleConfirm,
  };
}

// Usage: Simple, declarative
const { step, selectedSlots, handleSlotSelect } = useBookingFlow(bookableId);
```

**Strategic Benefits**: 
- Clean, testable interface
- No jQuery dependencies
- Type-safe booking flow
- Easy to modify and extend

### 7. Comments: What Code Can't Show

#### Good Comments: Explaining the "Why"
```typescript
// Good: Explains non-obvious logic
//hacky way to make sure that for example urls /<id>/edit/logo and /<id>/edit#logo are viewed as the same page
const normalizedPath = pathname.replace(/#.*$/, '').replace(/\/[^\/]+$/, '');
```

**Why it's good**: Explains the reasoning behind the hack, not just what it does.

#### Bad Comments: Restating the Code
```typescript
// Bad: Just restates the function name
// Returns true if user is logged in
function isUserLoggedIn(): boolean {
  return !!user;
}
```

**Why it's bad**: Comment doesn't add value beyond the function name.

#### Missing Comments: Hidden Invariants
```typescript
// Missing: No explanation of complex business logic
const steps = React.useMemo(
  () => {
    const hasQuestions =
      query.data?.questions && query.data.questions.length > 0;

    // Why do we need separate steps for booking vs questions?
    // What happens if questions are added/removed dynamically?
    // Why is validation schema built this way?

    const bookingStep = [
      /* ... */
    ];
    const questionsStep =
      hasQuestions &&
      {
        /* ... */
      };

    return [...bookingStep, ...(questionsStep ? [questionsStep] : [])];
  },
  [
    /* ... */
  ],
);
```

**Missing**: Comments explaining the business logic and design decisions.

---

## Key Takeaways for Our Codebase

### What We Did Wrong (Tactical Programming)
1. **Quick jQuery Integration**: Used events to bridge React and legacy code
2. **Monolithic Contexts**: Put everything in one place for immediate functionality
3. **Manual Schema Building**: Built validation schemas outside components
4. **Temporal Coupling**: Required specific initialization order without enforcement

### What Strategic Programming Looks Like
1. **Deep Modules**: Simple interfaces that hide complex implementations
2. **Information Hiding**: Components don't leak implementation details
3. **Declarative Flows**: State-driven instead of event-driven
4. **Focused Concerns**: Each module has a single responsibility

### The Investment Payoff
- **Short-term**: 10-20% slower initial development
- **Long-term**: 10-20% faster feature development
- **Quality**: Better code attracts better engineers
- **Maintainability**: Easier to onboard new developers
- **Reliability**: Fewer bugs and easier debugging


# Put it into practise. SurveyFormFlow Refactoring: From Shallow to Deep Module
**Applying "A Philosophy of Software Design" Principles**

## The Problem: Current SurveyFormFlow Component and Booking Component

### High Cognitive Load
The current `SurveyFormFlow.tsx` (490 lines) requires developers to understand multiple concerns simultaneously:

```typescript
// Current: Too many responsibilities in one component
function SurveyFormFlow({ message, showState, setShowSurveyResultsModal }) {
  // 1. Form validation schema generation
  const getSurveyQuestionValidationSchema = React.useCallback((question) => {
    switch (question.question_type) {
      case SurveyQuestionTypes.OPEN:
        return getOpenSurveyQuestionSchema(question, t);
      case SurveyQuestionTypes.SINGLE:
        return getSingleChoiceSurveyQuestionSchema(question, t);
      // ... 3 more cases
    }
  }, [t]);

  // 2. Multi-step form management
  const { currentStep, currentStepCount, formik, handleBack, maxSteps, isLastStep, isFirstStep, setCurrentStepCount } = useMultiStepForm({...});

  // 3. API submission logic with complex transformations
  const onSubmit = React.useCallback(async (values) => {
    const answers = await Promise.all(
      Object.entries(values).map(async ([questionId, value]) => {
        let apiAnswerObject;
        switch (value.answer_type) {
          case SurveyAnswerTypes.RATING:
            // Complex rating transformation
          case SurveyAnswerTypes.UPLOAD:
            // File upload logic
          case SurveyAnswerTypes.MULTI:
            // Multi-choice transformation
          default:
            // Default transformation
        }
        return { /* complex object */ };
      })
    );
    // More submission logic...
  }, [many, dependencies]);

  // 4. Privacy warning logic
  const shouldShowPrivacyWarning = account?.visibility === 'private' && survey.user_type === 'account' && survey.results_public === true && isFirstStep;

  // 5. Dirty form handling
  useDirtyFormConfirmPrompt(formik.dirty);

  // 6. UI rendering with complex conditional logic
  return (
    <SurveyFormModal>
      <MultiStepFormWrapper>
        {/* Complex conditional rendering */}
      </MultiStepFormWrapper>
    </SurveyFormModal>
  );
}
```

### Change Amplification
To modify survey behavior, you must touch:
1. **SurveyFormFlow.tsx** - Main component logic
2. **Multiple question schema files** - Validation logic
3. **Multiple initial value files** - Form setup logic
4. **API functions** - Submission logic
5. **Translation files** - UI text
6. **Type definitions** - Data structures

### Unknown Unknowns
- Hidden dependencies between form validation and question types
- Complex data transformation logic mixed with component logic
- File upload logic embedded in submission flow
- Privacy warning logic scattered throughout component

## The Solution: Deep Module Architecture

### 1. Create Focused Feature Module Structure

```
features/surveys/
├── components/
│   ├── SurveyFlow.tsx          # Simple interface component
│   ├── SurveyFormModal.tsx     # Modal wrapper
│   └── SurveyQuestion.tsx      # Individual question component
├── hooks/
│   ├── useSurveyFlow.ts        # Main survey logic
│   ├── useSurveyFormSetup.ts   # Form initialization
│   ├── useSurveySubmission.ts  # API submission logic
│   ├── useSurveyValidation.ts  # Validation schema generation
│   └── useSurveyPrivacy.ts     # Privacy warning logic
├── types/
│   ├── survey.types.ts         # All survey-related types
│   └── answers.types.ts        # Answer type definitions
├── utils/
│   ├── answerTransformers.ts   # Data transformation logic
│   └── questionHelpers.ts      # Question-specific utilities
└── index.ts                    # Public API
```

### 2. Deep Module: useSurveyFlow Hook

```typescript
// features/surveys/hooks/useSurveyFlow.ts
export function useSurveyFlow(message: SurveyMessage) {
  const { steps, initialValues } = useSurveyFormSetup(message);
  const { submitSurvey, isSubmitting } = useSurveySubmission(message);
  const { shouldShowPrivacyWarning } = useSurveyPrivacy(message);
  
  const formFlow = useMultiStepForm({
    steps,
    initialValues,
    onSubmit: submitSurvey,
  });
  
  return {
    // Simple interface - hides all complexity
    formFlow,
    shouldShowPrivacyWarning,
    isSubmitting,
  };
}
```

### 3. Focused Hook: useSurveyFormSetup

```typescript
// features/surveys/hooks/useSurveyFormSetup.ts
export function useSurveyFormSetup(message: SurveyMessage) {
  const { t } = useTranslation();
  const query = useSurveySubmissionQuery(message);
  
  const steps = React.useMemo(() => {
    return message.survey.questions.map((question) => ({
      component: <SurveyFormQuestion question={question} />,
      validationSchema: getQuestionValidationSchema(question, t),
    }));
  }, [message.survey.questions, t]);
  
  const initialValues = React.useMemo(() => {
    return message.survey.questions.reduce((acc, question) => {
      acc[question.id] = getQuestionInitialValue(question, query.data);
      return acc;
    }, {} as SurveyFormAnswers);
  }, [message.survey.questions, query.data]);
  
  return { steps, initialValues };
}
```

### 4. Focused Hook: useSurveySubmission

```typescript
// features/surveys/hooks/useSurveySubmission.ts
export function useSurveySubmission(message: SurveyMessage) {
  const queryClient = useQueryClient();
  
  const { mutateAsync: submitAnswers, isPending: isSubmitting } = useMutation({
    mutationFn: async (values: SurveyFormAnswers) => {
      const answers = await transformAnswersForAPI(values, message.in_group_id);
      
      if (message.survey.submission_id) {
        return editSurveyAnswers(message.id, message.survey.submission_id, answers);
      }
      
      return submitSurveyAnswers(message.id, answers);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['messages', message.id] });
    },
  });
  
  return { submitSurvey: submitAnswers, isSubmitting };
}
```

### 5. Utility: Answer Transformers

```typescript
// features/surveys/utils/answerTransformers.ts
export async function transformAnswersForAPI(
  values: SurveyFormAnswers,
  groupId: string
): Promise<SurveyFormAnswer[]> {
  return Promise.all(
    Object.entries(values).map(async ([questionId, value]) => {
      const transformer = getAnswerTransformer(value.answer_type);
      const apiAnswerObject = await transformer(value, groupId);
      
      return {
        question_id: questionId,
        answer_type: value.answer_type,
        other_option_value: value.other_option_value,
        ...apiAnswerObject,
      };
    })
  );
}

function getAnswerTransformer(answerType: SURVEY_ANSWER_TYPES) {
  switch (answerType) {
    case SurveyAnswerTypes.RATING:
      return transformRatingAnswer;
    case SurveyAnswerTypes.UPLOAD:
      return transformUploadAnswer;
    case SurveyAnswerTypes.MULTI:
      return transformMultiChoiceAnswer;
    default:
      return transformDefaultAnswer;
  }
}
```

### 6. Deep Module: SurveyFlow Component

```typescript
// features/surveys/components/SurveyFlow.tsx
export function SurveyFlow({ 
  message, 
  showState, 
  setShowSurveyResultsModal 
}: SurveyFlowProps) {
  const { formFlow, shouldShowPrivacyWarning, isSubmitting } = useSurveyFlow(message);
  
  // Simple interface - complex implementation hidden
  return (
    <SurveyFormModal showState={showState}>
      <MultiStepFormWrapper
        formik={formFlow.formik}
        currentStep={
          <>
            {shouldShowPrivacyWarning && <PrivacyWarning />}
            {formFlow.currentStep.component}
          </>
        }
        header={
          <MultiStepFormProgressHeader
            title={message.title}
            currentStepCount={formFlow.currentStepCount}
            maxSteps={formFlow.maxSteps}
          />
        }
        footer={
          <SurveyFormFooter
            formFlow={formFlow}
            isSubmitting={isSubmitting}
            onClose={showState.setShow}
          />
        }
      />
      <SurveySubmissionConfirmationModal
        message={message}
        setShowSurveyResultsModal={setShowSurveyResultsModal}
      />
    </SurveyFormModal>
  );
}
```

### 7. Public API

```typescript
// features/surveys/index.ts
export { SurveyFlow } from './components/SurveyFlow';
export { useSurveyFlow } from './hooks/useSurveyFlow';
export type { SurveyMessage, SurveyFormAnswers } from './types/survey.types';

// Usage: Simple and focused
import { SurveyFlow } from '@/features/surveys';

<SurveyFlow 
  message={message}
  showState={{ show, setShow }}
  setShowSurveyResultsModal={setShowSurveyResultsModal}
/>
```

## Benefits of the Refactored Architecture

### 1. Reduced Cognitive Load
**Before**: Developer must understand 6+ concerns in one file
**After**: Developer only needs to understand the specific hook they're working on

### 2. Eliminated Change Amplification
**Before**: Survey changes touch 6+ files across different concerns
**After**: Survey changes are contained within the `features/surveys/` module

### 3. Deep Modules
- **SurveyFlow**: Simple interface (`message`, `showState`, `setShowSurveyResultsModal`)
- **useSurveyFlow**: Hides all complex survey logic behind simple return object
- **Answer Transformers**: Encapsulate complex data transformation logic

### 4. Information Hiding
- File upload logic hidden in `transformUploadAnswer`
- Validation schema generation hidden in `useSurveyFormSetup`
- Privacy warning logic hidden in `useSurveyPrivacy`

### 5. Single Responsibility
- Each hook handles one specific concern
- Each utility function has one clear purpose
- Components focus on rendering, not business logic

## Success Metrics

### Development Velocity
- **Change Amplification**: Survey changes touch 1-2 files instead of 6+
- **Cognitive Load**: New developers understand survey logic in hours instead of days
- **Unknown Unknowns**: Clear module boundaries eliminate hidden dependencies

### Code Quality
- **Module Depth**: Simple interfaces with complex implementations hidden
- **Information Hiding**: No implementation details leaked to consumers
- **Single Responsibility**: Each module has one clear purpose

## Conclusion

This refactoring transforms a **shallow module** (complex interface, mixed responsibilities) into **deep modules** (simple interfaces, focused responsibilities) that follow the principles from "A Philosophy of Software Design":

1. **Deep Modules**: Simple interfaces that hide complex implementations
2. **Information Hiding**: Implementation details are encapsulated within modules
3. **Reduced Complexity**: Developers only face a small fraction of complexity at any time
4. **Strategic Programming**: Investment in good design pays off in long-term productivity

The result is a survey system that's easier to understand, modify, and maintain, with clear boundaries and reduced coupling between different parts of the system.

