---
trigger: always_on
description: Form handling patterns with React Hook Form and Zod validation
---


# Form Handling Rules

## React Hook Form + Zod Pattern

### Basic Form Setup
```tsx
// components/forms/UserForm.tsx
interface UserFormProps {
  onSubmit: (data: UserFormData) => void
  initialData?: Partial<User>
  isLoading?: boolean
}

export const UserForm = ({ onSubmit, initialData, isLoading }: UserFormProps) => {
  const form = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: initialData?.name || '',
      email: initialData?.email || '',
      phone: initialData?.phone || '',
    }
  })

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Full Name</FormLabel>
              <FormControl>
                <Input placeholder="Enter full name" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input type="email" placeholder="Enter email" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        
        <Button type="submit" disabled={isLoading} className="w-full">
          {isLoading ? 'Saving...' : 'Save User'}
        </Button>
      </form>
    </Form>
  )
}
```

### Form Hook Pattern
```tsx
// hooks/useUserForm.ts
export const useUserForm = (userId?: string) => {
  const { data: user } = useUser(userId)
  const createUser = useCreateUser()
  const updateUser = useUpdateUser()
  
  const form = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
      phone: '',
    }
  })
  
  // Reset form when user data loads
  useEffect(() => {
    if (user) {
      form.reset({
        name: user.name,
        email: user.email,
        phone: user.phone || '',
      })
    }
  }, [user, form])
  
  const handleSubmit = (data: UserFormData) => {
    if (userId) {
      updateUser.mutate({ id: userId, data })
    } else {
      createUser.mutate(data)
    }
  }
  
  const isLoading = createUser.isPending || updateUser.isPending
  
  return {
    form,
    handleSubmit,
    isLoading,
    reset: form.reset,
  }
}
```

### Advanced Form Patterns

#### Multi-Step Form
```tsx
// hooks/useMultiStepForm.ts
export const useMultiStepForm = <T extends Record<string, any>>(
  steps: Array<{ key: string; schema: ZodSchema<any> }>,
  onComplete: (data: T) => void
) => {
  const [currentStep, setCurrentStep] = useState(0)
  const [formData, setFormData] = useState<Partial<T>>({})
  
  const currentStepConfig = steps[currentStep]
  
  const form = useForm({
    resolver: zodResolver(currentStepConfig.schema),
    defaultValues: formData[currentStepConfig.key] || {}
  })
  
  const nextStep = (data: any) => {
    setFormData(prev => ({ ...prev, [currentStepConfig.key]: data }))
    
    if (currentStep < steps.length - 1) {
      setCurrentStep(prev => prev + 1)
    } else {
      onComplete({ ...formData, [currentStepConfig.key]: data } as T)
    }
  }
  
  const prevStep = () => {
    if (currentStep > 0) {
      setCurrentStep(prev => prev - 1)
    }
  }
  
  return {
    form,
    currentStep,
    totalSteps: steps.length,
    isFirstStep: currentStep === 0,
    isLastStep: currentStep === steps.length - 1,
    nextStep,
    prevStep,
    handleSubmit: form.handleSubmit(nextStep)
  }
}
```

#### Dynamic Form Fields
```tsx
// components/forms/DynamicFieldArray.tsx
export const DynamicFieldArray = ({ name, control }: DynamicFieldArrayProps) => {
  const { fields, append, remove } = useFieldArray({
    control,
    name,
  })

  return (
    <div className="space-y-4">
      {fields.map((field, index) => (
        <div key={field.id} className="flex gap-2 items-end">
          <FormField
            control={control}
            name={`${name}.${index}.value`}
            render={({ field }) => (
              <FormItem className="flex-1">
                <FormLabel>Item {index + 1}</FormLabel>
                <FormControl>
                  <Input {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <Button
            type="button"
            variant="outline"
            size="icon"
            onClick={() => remove(index)}
          >
            <X className="h-4 w-4" />
          </Button>
        </div>
      ))}
      
      <Button
        type="button"
        variant="outline"
        onClick={() => append({ value: '' })}
      >
        <Plus className="h-4 w-4 mr-2" />
        Add Item
      </Button>
    </div>
  )
}
```

### Form Validation Rules

#### Complex Validation Schema
```tsx
// lib/validations/userProfile.ts
export const userProfileSchema = z.object({
  personal: z.object({
    firstName: z.string().min(2, 'First name required'),
    lastName: z.string().min(2, 'Last name required'),
    email: emailSchema,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chihebnabil/lovable-boilerplate](https://github.com/chihebnabil/lovable-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
