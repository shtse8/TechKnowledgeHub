# Model-View-ViewModel (MVVM)

An architectural pattern that facilitates the separation of the development of the graphical user interface from the business logic or back-end logic, enhancing testability and maintainability.

## Historical Context

**Origin**: Developed by Microsoft architects Ken Cooper and Ted Peters in 2005 as a variation of the Presentation Model pattern for Windows Presentation Foundation (WPF).

**Evolution**: Gained widespread adoption in modern UI frameworks and libraries like Angular, Knockout.js, Vue.js, and Swift UI. Became particularly important with the rise of reactive programming and data binding technologies.

## Core Concepts

1. **Model**:
   - Represents the data and business logic
   - Contains domain-specific classes and data validation
   - Independent of the user interface
   - Often includes data access and business rules

2. **View**:
   - Represents the UI elements (what the user sees)
   - Passive; has no knowledge of the Model
   - Binds to properties exposed by the ViewModel
   - Forwards user interactions to the ViewModel via commands

3. **ViewModel**:
   - Acts as an intermediary between the View and Model
   - Exposes data from the Model in a View-friendly format
   - Contains presentation logic but no direct references to UI elements
   - Implements properties and commands the View can bind to

4. **Data Binding**:
   - Automatic synchronization between View and ViewModel
   - Changes in the ViewModel are reflected in the View
   - User inputs in the View update the ViewModel
   - Key mechanism that enables loose coupling

## Implementation

### Key Components

1. **Data Binding Mechanism**:
   - One-way binding: ViewModel to View updates only
   - Two-way binding: Changes propagate in both directions
   - Event binding: Maps user actions to ViewModel commands

2. **Commands**:
   - Encapsulates actions or operations
   - Often implemented as objects with execute and can-execute logic
   - Maps directly to user interactions (clicks, inputs, etc.)

3. **Observable Properties**:
   - Properties in the ViewModel that notify when their values change
   - Enables the View to update when Model data changes
   - Often implemented using observer pattern or reactive extensions

4. **State Management**:
   - Manages UI state independent of business logic
   - Handles view-specific concerns like loading states, error messages
   - Coordinates multiple views or view components

## Example

### WPF Application (C#)

```csharp
// MODEL
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime DateOfBirth { get; set; }
    
    public int Age
    {
        get
        {
            return DateTime.Today.Year - DateOfBirth.Year - 
                   (DateTime.Today.DayOfYear < DateOfBirth.DayOfYear ? 1 : 0);
        }
    }
}

public interface ICustomerService
{
    Task<Customer> GetCustomerByIdAsync(int id);
    Task<bool> UpdateCustomerAsync(Customer customer);
}

// VIEW-MODEL
public class CustomerViewModel : INotifyPropertyChanged
{
    private readonly ICustomerService _customerService;
    private Customer _customer;
    private bool _isLoading;
    private string _errorMessage;
    
    // Implement INotifyPropertyChanged for data binding
    public event PropertyChangedEventHandler PropertyChanged;
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
    
    // Properties exposed to the View
    public int Id 
    { 
        get => _customer?.Id ?? 0; 
        set
        {
            if (_customer != null && _customer.Id != value)
            {
                _customer.Id = value;
                OnPropertyChanged();
            }
        }
    }
    
    public string Name 
    { 
        get => _customer?.Name ?? string.Empty; 
        set
        {
            if (_customer != null && _customer.Name != value)
            {
                _customer.Name = value;
                OnPropertyChanged();
                // Example of command that depends on this property
                SaveCommand.RaiseCanExecuteChanged();
            }
        }
    }
    
    public string Email 
    { 
        get => _customer?.Email ?? string.Empty; 
        set
        {
            if (_customer != null && _customer.Email != value)
            {
                _customer.Email = value;
                OnPropertyChanged();
                SaveCommand.RaiseCanExecuteChanged();
            }
        }
    }
    
    public DateTime DateOfBirth 
    { 
        get => _customer?.DateOfBirth ?? DateTime.Today; 
        set
        {
            if (_customer != null && _customer.DateOfBirth != value)
            {
                _customer.DateOfBirth = value;
                OnPropertyChanged();
                OnPropertyChanged(nameof(Age)); // Age depends on DateOfBirth
            }
        }
    }
    
    public int Age => _customer?.Age ?? 0;
    
    public bool IsLoading
    {
        get => _isLoading;
        private set
        {
            if (_isLoading != value)
            {
                _isLoading = value;
                OnPropertyChanged();
                LoadCommand.RaiseCanExecuteChanged();
                SaveCommand.RaiseCanExecuteChanged();
            }
        }
    }
    
    public string ErrorMessage
    {
        get => _errorMessage;
        private set
        {
            if (_errorMessage != value)
            {
                _errorMessage = value;
                OnPropertyChanged();
            }
        }
    }
    
    // Commands bound to the View
    public RelayCommand LoadCommand { get; private set; }
    public RelayCommand SaveCommand { get; private set; }
    
    public CustomerViewModel(ICustomerService customerService)
    {
        _customerService = customerService;
        _customer = new Customer();
        
        // Initialize commands
        LoadCommand = new RelayCommand(
            async () => await LoadCustomerAsync(),
            () => !IsLoading
        );
        
        SaveCommand = new RelayCommand(
            async () => await SaveCustomerAsync(),
            () => !IsLoading && !string.IsNullOrEmpty(Name) && !string.IsNullOrEmpty(Email)
        );
    }
    
    private async Task LoadCustomerAsync()
    {
        try
        {
            IsLoading = true;
            ErrorMessage = string.Empty;
            
            _customer = await _customerService.GetCustomerByIdAsync(Id);
            
            // Notify UI of all property changes
            OnPropertyChanged(nameof(Id));
            OnPropertyChanged(nameof(Name));
            OnPropertyChanged(nameof(Email));
            OnPropertyChanged(nameof(DateOfBirth));
            OnPropertyChanged(nameof(Age));
        }
        catch (Exception ex)
        {
            ErrorMessage = $"Error loading customer: {ex.Message}";
        }
        finally
        {
            IsLoading = false;
        }
    }
    
    private async Task SaveCustomerAsync()
    {
        try
        {
            IsLoading = true;
            ErrorMessage = string.Empty;
            
            bool success = await _customerService.UpdateCustomerAsync(_customer);
            
            if (!success)
            {
                ErrorMessage = "Failed to update customer.";
            }
        }
        catch (Exception ex)
        {
            ErrorMessage = $"Error saving customer: {ex.Message}";
        }
        finally
        {
            IsLoading = false;
        }
    }
}

// Helper class for commands
public class RelayCommand : ICommand
{
    private readonly Func<Task> _execute;
    private readonly Func<bool> _canExecute;
    private bool _isExecuting;
    
    public event EventHandler CanExecuteChanged;
    
    public RelayCommand(Func<Task> execute, Func<bool> canExecute = null)
    {
        _execute = execute;
        _canExecute = canExecute;
    }
    
    public bool CanExecute(object parameter)
    {
        return !_isExecuting && (_canExecute == null || _canExecute());
    }
    
    public async void Execute(object parameter)
    {
        if (CanExecute(parameter))
        {
            try
            {
                _isExecuting = true;
                RaiseCanExecuteChanged();
                await _execute();
            }
            finally
            {
                _isExecuting = false;
                RaiseCanExecuteChanged();
            }
        }
    }
    
    public void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}

// VIEW (XAML)
/*
<Window x:Class="MVVMExample.CustomerView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Customer Details" Width="400" Height="350">
    <Grid Margin="10">
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        
        <!-- Customer ID -->
        <Label Grid.Row="0" Grid.Column="0" Content="ID:"/>
        <TextBox Grid.Row="0" Grid.Column="1" Text="{Binding Id, UpdateSourceTrigger=PropertyChanged}"/>
        
        <!-- Load Button -->
        <Button Grid.Row="1" Grid.Column="1" Content="Load Customer" 
                Command="{Binding LoadCommand}" HorizontalAlignment="Left" Margin="0,5,0,10"/>
        
        <!-- Customer Name -->
        <Label Grid.Row="2" Grid.Column="0" Content="Name:"/>
        <TextBox Grid.Row="2" Grid.Column="1" Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}"/>
        
        <!-- Customer Email -->
        <Label Grid.Row="3" Grid.Column="0" Content="Email:"/>
        <TextBox Grid.Row="3" Grid.Column="1" Text="{Binding Email, UpdateSourceTrigger=PropertyChanged}"/>
        
        <!-- Customer Date of Birth -->
        <Label Grid.Row="4" Grid.Column="0" Content="Date of Birth:"/>
        <DatePicker Grid.Row="4" Grid.Column="1" SelectedDate="{Binding DateOfBirth}"/>
        
        <!-- Customer Age (read-only) -->
        <Label Grid.Row="5" Grid.Column="0" Content="Age:"/>
        <TextBlock Grid.Row="5" Grid.Column="1" Text="{Binding Age}" VerticalAlignment="Center"/>
        
        <!-- Save Button -->
        <Button Grid.Row="6" Grid.Column="1" Content="Save Changes" 
                Command="{Binding SaveCommand}" HorizontalAlignment="Left" VerticalAlignment="Top" Margin="0,10,0,0"/>
        
        <!-- Loading Indicator -->
        <TextBlock Grid.Row="6" Grid.Column="1" HorizontalAlignment="Right" VerticalAlignment="Top" Margin="0,10,0,0">
            <TextBlock.Style>
                <Style TargetType="TextBlock">
                    <Setter Property="Text" Value=""/>
                    <Style.Triggers>
                        <DataTrigger Binding="{Binding IsLoading}" Value="True">
                            <Setter Property="Text" Value="Loading..."/>
                        </DataTrigger>
                    </Style.Triggers>
                </Style>
            </TextBlock.Style>
        </TextBlock>
        
        <!-- Error Message -->
        <TextBlock Grid.Row="6" Grid.Column="1" Text="{Binding ErrorMessage}" 
                   Foreground="Red" VerticalAlignment="Bottom"/>
    </Grid>
</Window>
*/
```

### Angular Application (TypeScript)

```typescript
// MODEL
export interface Customer {
  id: number;
  name: string;
  email: string;
  dateOfBirth: Date;
}

// Service (interacts with the Model)
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';
import { Customer } from './customer.model';

@Injectable({
  providedIn: 'root'
})
export class CustomerService {
  private apiUrl = 'api/customers';
  
  constructor(private http: HttpClient) {}
  
  getCustomer(id: number): Observable<Customer> {
    return this.http.get<Customer>(`${this.apiUrl}/${id}`)
      .pipe(
        catchError(error => throwError(`Error loading customer: ${error.message}`))
      );
  }
  
  updateCustomer(customer: Customer): Observable<Customer> {
    return this.http.put<Customer>(`${this.apiUrl}/${customer.id}`, customer)
      .pipe(
        catchError(error => throwError(`Error updating customer: ${error.message}`))
      );
  }
}

// VIEW-MODEL (Angular Component)
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Customer } from './customer.model';
import { CustomerService } from './customer.service';

@Component({
  selector: 'app-customer-detail',
  templateUrl: './customer-detail.component.html',
  styleUrls: ['./customer-detail.component.css']
})
export class CustomerDetailComponent implements OnInit {
  customerForm: FormGroup;
  customer: Customer;
  loading = false;
  errorMessage = '';
  
  constructor(
    private fb: FormBuilder,
    private customerService: CustomerService
  ) {}
  
  ngOnInit(): void {
    // Initialize form (ViewModel properties)
    this.customerForm = this.fb.group({
      id: [0, Validators.required],
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      dateOfBirth: [null, Validators.required]
    });
  }
  
  // Computed property (presentation logic)
  get age(): number {
    const birthDate = this.customerForm.get('dateOfBirth').value;
    if (!birthDate) return 0;
    
    const today = new Date();
    let age = today.getFullYear() - birthDate.getFullYear();
    const monthDiff = today.getMonth() - birthDate.getMonth();
    
    if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthDate.getDate())) {
      age--;
    }
    
    return age;
  }
  
  // Command methods
  loadCustomer(): void {
    const id = this.customerForm.get('id').value;
    if (!id) return;
    
    this.loading = true;
    this.errorMessage = '';
    
    this.customerService.getCustomer(id).subscribe(
      customer => {
        this.customer = customer;
        this.customerForm.patchValue({
          name: customer.name,
          email: customer.email,
          dateOfBirth: customer.dateOfBirth
        });
        this.loading = false;
      },
      error => {
        this.errorMessage = error;
        this.loading = false;
      }
    );
  }
  
  saveCustomer(): void {
    if (this.customerForm.invalid) return;
    
    this.loading = true;
    this.errorMessage = '';
    
    const customer: Customer = {
      id: this.customerForm.get('id').value,
      name: this.customerForm.get('name').value,
      email: this.customerForm.get('email').value,
      dateOfBirth: this.customerForm.get('dateOfBirth').value
    };
    
    this.customerService.updateCustomer(customer).subscribe(
      updatedCustomer => {
        this.customer = updatedCustomer;
        this.loading = false;
      },
      error => {
        this.errorMessage = error;
        this.loading = false;
      }
    );
  }
}

// VIEW (HTML Template)
/*
<div class="customer-container">
  <form [formGroup]="customerForm">
    <div class="form-group">
      <label for="id">ID:</label>
      <div class="input-group">
        <input id="id" type="number" formControlName="id" class="form-control">
        <div class="input-group-append">
          <button 
            type="button" 
            class="btn btn-primary" 
            (click)="loadCustomer()" 
            [disabled]="loading || !customerForm.get('id').valid">
            Load
          </button>
        </div>
      </div>
    </div>
    
    <div class="form-group">
      <label for="name">Name:</label>
      <input id="name" type="text" formControlName="name" class="form-control">
      <div *ngIf="customerForm.get('name').invalid && customerForm.get('name').touched" class="error-message">
        Name is required
      </div>
    </div>
    
    <div class="form-group">
      <label for="email">Email:</label>
      <input id="email" type="email" formControlName="email" class="form-control">
      <div *ngIf="customerForm.get('email').invalid && customerForm.get('email').touched" class="error-message">
        Valid email is required
      </div>
    </div>
    
    <div class="form-group">
      <label for="dob">Date of Birth:</label>
      <input id="dob" type="date" formControlName="dateOfBirth" class="form-control">
    </div>
    
    <div class="form-group">
      <label>Age:</label>
      <span>{{ age }}</span>
    </div>
    
    <div class="form-actions">
      <button 
        type="button" 
        class="btn btn-success" 
        (click)="saveCustomer()" 
        [disabled]="loading || customerForm.invalid">
        Save
      </button>
      <span *ngIf="loading" class="ml-2">Loading...</span>
    </div>
    
    <div *ngIf="errorMessage" class="alert alert-danger mt-3">
      {{ errorMessage }}
    </div>
  </form>
</div>
*/
```

## Advantages and Limitations

### Advantages
- **Separation of Concerns**: Clear division between UI and business logic
- **Testability**: ViewModel can be tested independently of the View
- **Design-Development Workflow**: Designers can work on Views while developers work on ViewModels
- **Data Binding**: Reduces boilerplate UI update code
- **Reusability**: Same ViewModel can be used with different Views

### Limitations
- **Complexity**: Additional layer can be overkill for simple applications
- **Learning Curve**: Data binding and command patterns add conceptual complexity
- **Performance**: Extensive data binding can impact performance in complex UIs
- **Over-abstraction**: Can lead to abstract ViewModels that are hard to understand
- **Framework Dependency**: Often relies on specific framework features

## Related Concepts
- Model-View-Controller (MVC)
- Model-View-Presenter (MVP)
- Presentation Model
- Data Binding
- Reactive Programming 