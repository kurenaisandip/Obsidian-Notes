
https://www.youtube.com/watch?v=WSBy_Ypgk38&t=0s




# Video -2
so far we have  the base of the mvp pattern. The application still doesnot work since the repositories of the data access layer need to be implemented.
We can add a new project for the data access layer or just add a folder as a component fo rthe repositories like will do now. Create new folder name Repositories.
so, we add a class for the base items of the repository objects.

Repository/BaseRepository.cs

we convert to an abstract class so that it can only be used through inheritance
here we will only define a protected field for the connection string. We can define more common fields and methods.

    public abstract class BaseRepository
    {
        protected string connectionString;
        //...
    }

for repository objects okay now we will add the pet repository concrete class then inherit the base repository class and reference the data and sql client namespace 

![[Pasted image 20240309215526.png]]

then we implement the pet repository interface defined in the model.

![[Pasted image 20240309215637.png]]


all right now we'll implement the get all and get by value method for searches but before that we'll define the constructor. the constructor will accept a parameter for the connection string then we assign it to the connection string field of the base repository class. this is optional you can initialize the connection string directly in the base class however to keep the code clean and testable it is recommended to do it this way for reasons of dependency injection and unit testing.
![[Pasted image 20240309215851.png]]

okay now we implement the get all method. first we define a list of pet models to store the result and return the list then we query the database so using the using statement we create
the sql connection instance from the connection string for communication with the database
in the same way using another using statement we create the sql command to execute the operations on the database. we open the connection to sql server and establish the connection to the command to execute the operations. we set the text command in this case we select everything from the pets table. optionally we sort on the id column in descending order to show the last recorded data first then using another using statement we run the command reader note that the using statement causes objects to be disposed of correctly so there is no need to close the connection or the reader now while the reader is reading the rows. we create a pet model instance then assign the cell's value to model properties make sure to cast to the appropriate data type and finally we add the object to the list of pets.

    public class PetRepository : BaseRepository, IPetRepository
    {
    
        //Methods
        public IEnumerable<PetModel> GetAll()
        {
            var petList = new List<PetModel>();
            using (var connection = new SqlConnection(connectionString))
            using (var command = new SqlCommand())
            {
                connection.Open();
                command.Connection = connection;
                command.CommandText = "Select *from Pet order by Pet_Id desc";
                using (var reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        var petModel = new PetModel();
                        petModel.Id = (int)reader[0];
                        petModel.Name = reader[1].ToString();
                        petModel.Type = reader[2].ToString();
                        petModel.Colour = reader[3].ToString();
                        petList.Add(petModel);
                    }
                }
            }
            return petList;
        }

    }




we do similarly in the get by value
method. we modify the query to search for the pet by id or to search for the pet by name in this case we will use the logical operator like to search for the name by matches that is to find all the names of pets that start with the value search okay now we declare the id parameter and the name parameter since we only have one search value we need to determine if the value is numeric or letters otherwise an error will be thrown so here we will define a local field of type integer for the value of the id, parameter we do similarly for the name parameter now we define the local search fields, so we define a field of type integer for the pet id and we assign the value of the search parameter of the method but as long as it is numeric for this we try to convert the search value to an integer if the value can be converted. we set the search value otherwise we set the value to zero finally and optionally we define the search field by name here it is not necessary to do any conversion however it would be better to define two text commands to speed up the query all right now the form should be able to display the list of pets and do the searches.

    public class PetRepository : BaseRepository, IPetRepository
    {
        //Constructor
        public PetRepository(string connectionString)
        {
            this.connectionString = connectionString;
        }
        //Methods
       
        public IEnumerable<PetModel> GetByValue(string value)
        {
            var petList = new List<PetModel>();
            int petId = int.TryParse(value, out _) ? Convert.ToInt32(value) : 0;
            string petName = value;
            using (var connection = new SqlConnection(connectionString))
            using (var command = new SqlCommand())
            {
                connection.Open();
                command.Connection = connection;
                command.CommandText = @"Select *from Pet
                                        where Pet_Id=@id or Pet_Name like  @name+'%' 
                                        order by Pet_Id desc";
                command.Parameters.Add("@id", SqlDbType.Int).Value = petId;
                command.Parameters.Add("@name", SqlDbType.NVarChar).Value = petName;

                using (var reader = command.ExecuteReader())
                {
                    while (reader.Read())
                    {
                        var petModel = new PetModel();
                        petModel.Id = (int)reader[0];
                        petModel.Name = reader[1].ToString();
                        petModel.Type = reader[2].ToString();
                        petModel.Colour = reader[3].ToString();
                        petList.Add(petModel);
                    }
                }
            }
            return petList;
        }
        public void Add(PetModel petModel)
        {
            throw new NotImplementedException();
        }
        public void Delete(int id)
        {
            throw new NotImplementedException();
        }
        public void Edit(PetModel petModel)
        {
            throw new NotImplementedException();
        }


    }





so, let's run it to do this we go to the program class we add the references to all the components well so that the application works we need to instantiate the presenter and inject the concrete class of the view and repository or model.

![[Pasted image 20240309221044.png]]

we can do it directly on a single line as in this
example but i will define it in local fields so that it looks a little organized so we define and instantiate the view similarly for the repository.

![[Pasted image 20240309221218.png]]


the repository requires us to inject the connection string so we also define and initialize a field for it. we can set the connection string directly in this field 
![[Pasted image 20240309221358.png]]

or we can do it in the app configuration file well it is recommended to define any configuration in this file it is standard practice to do so so that you have all the configuration values in one place. if you have experience with the markup language you can directly write the connection string here otherwise you can do it from the project properties like. now will now we open the settings section. well we assign a name we establish the type of configuration in this case connection string. in scope we select application finally we establish the connection string value we can write it or we can use the properties window. we put the name or ip of the server in my case local we choose the type of authentication if it is with windows or with username and password finally we select the database in my case veterinary database. optionally we test if the connection string works correctly.
![[Pasted image 20240309221527.png]]

well for security we save the changes and return to the configuration file to verify that everything is correct okay here's the connection string section and also the connection string. we created just now you can edit the value from here or replicate and edit it
for other connection strings.
![[Pasted image 20240309221719.png]]

Similarly, we can change the name of the connection string as it is very long. We copy the name and return to the program class, so we set the connection string from the app configurations. This requires adding the reference to the configuration assembly.
![[Pasted image 20240309222211.png]]
![[Pasted image 20240309222237.png]]


When we select the connection strings from the configuration manager, we specify the name of the connection string we want to access and get the connection string.

All right, now we instantiate the presenter and inject the view and repository objects into it.

Finally, we run the view. The list of pets is displayed correctly, and the search by id and name works correctly.
![[Pasted image 20240309222431.png]]

![[Pasted image 20240309222454.png]]



We had said that dependency injection deals with how an object meets another dependent object through the interface. Well, that's what we did here. The presenter accepts as parameters the interfaces that implement the view and repository, and here the presenter instance is created and we are injecting it with the concrete class of the view and repository. In such a way that the presenter can operate and manipulate them through the interface without directly depending on them. 

![[Pasted image 20240309222629.png]]

The dependency inversion principle should not be confused with dependency injection; they are different things. One is a principle, and the other is a pattern. Although there is a relationship, and they play an important role between the two.

![[Pasted image 20240309222701.png]]

Well, going back to the concepts, dependency injection allows you to better manage future code changes and application complexity, which helps make code reusable, maintainable, and loosely coupled between classes.

There are some frameworks that make it easier for us to do dependency injection. For example, Unity, Spring, StructureMap, and so forth.

In this way, it would be to do the dependency injection explicitly or manually.
![[Pasted image 20240309222816.png]]

instead of using a framework. For example, with Unity, 
![[Pasted image 20240309222846.png]]

it would be this way: the type mapping in the container is registered so that it can create the correct object for the given type that is set, which class should be instantiated for that specified interface. The `resolve` method takes care of creating the object and automatically injects the dependencies. Also, we would have the type assignment in a single container and place. This would save us a lot of time if we have tens or hundreds of classes and interfaces. Maybe I'll do a tutorial on this.

Well, for the tutorial to be complete, we will add a main view and a main presenter to do the dependency injection of the other elements since obviously, the application will have other models and forms, for example, pet owner, medicines, food, branches, and so forth. So, it would not be fair to leave it that way.

So, we add the interface for the main view. 
![[Pasted image 20240309222953.png]]

Here we define all the events to open the child forms, for example, show the view of pet owners, vets, and so forth.

Now, we add a form for the concrete main view.
    public interface IMainView
    {
        event EventHandler ShowPetView;
        event EventHandler ShowOwnerView;
        event EventHandler ShowVetsView;
    }

We design the main form as we like.
	![[Pasted image 20240309223137.png]]

Once the design is finished, we go to the code. We implement the interface for the main view.

Okay, now, in the same way that we did in the pets view, we associate and generate the events.

![[Pasted image 20240309223323.png]]

Finally, we added the main presenter. We do the same thing we did in the pet presenter: add the references to the view model and repository component.

Then, we define the following fields: a private field for the view using the main view interface. We will also define a read-only field for the connection string.

We then generate the constructor with parameters for the view and the connection string. We subscribe the handler event methods to view events. In this case, just the event of showing the pet view. Finally, we implement the show pet view method.

We would do the same with the other views and repositories or models.

![[Pasted image 20240309223523.png]]

All right, so we go back to the program class and modify it to run the main view and the presenter 
![[Pasted image 20240309223616.png]]

Great, everything is fine.

We always have a common inconvenience that the same form can be opened as many times as we want,
![[Pasted image 20240309223737.png]]
although sometimes this is required, but generally, it is only necessary to open a single form. Well, we have ways to prevent this. The first is through a loop: we go through the list of all open forms and check if the form is open or not. If not, we create the instance; otherwise, we just redisplay the form. Second is to use the singleton pattern. 
![[Pasted image 20240309223820.png]]

This pattern restricts the instantiation of a class to a single instance, which will allow a single form to be opened. And since we're on the topic of patterns and best practices, we'll do it through the singleton pattern. To do this, we go to the code of the pet view form.

*PetView.cs*

First, for the form instance, we define a static field of the same class type, in this case, pet view.

Now, we create a static method to get the form instance. We define a condition: if the form instance is equal to null or is disposed, we create the form instance. Otherwise, if the form instance already exists, we simply bring the form to the front to display it. We define another condition to restore the form in case it is minimized. Finally, we return the form instance.

![[Pasted image 20240309224113.png]]



And that's it. Now we go to the main presenter and modify the view creation. Here the get instance method would be used to create the instance.

![[Pasted image 20240309224214.png]]

Okay, the form is opened only once, and it is shown again in front in case it is left behind.



![[Pasted image 20240309224340.png]]

now we will need to set the main view as the mdi parent of the pet view there are many ways to do this too. but we can take advantage of the get instance method of the child form. so, this method will accept a parameter of type form as the parent container. now when the child form is instantiated. we set the parent containers parameters to the mdi property of the child form.

![[Pasted image 20240309225558.png]]

well we go back to the main presenter and when getting the instance of the pet view we send the main view as the parent container. we convert the form type or main view in both cases it will work. since the main view inherits from form 

![[Pasted image 20240309225638.png]]


Great, but now it's missing a close button, so we again added. 

Well, if we want, we can also make the child form open to the entire container and without borders. We remove the border style and set the dock property to fill.

![[Pasted image 20240309225724.png]]

All right, now we'll also make the form responsive by resizing the main form. To do this, we simply set the anchor or dock property properly as needed.

![[Pasted image 20240309225822.png]]

Well, that's all for now. We already have the fully functional application. Now we just need to implement the add, edit, and delete options. We will do that in the next and last video. Well, until next time.

# 3rd video- Implementing edit, add and delete

Let's continue with the third and final part of the tutorial.

Now we will implement the add, edit, and delete functions.

First, we will associate and generate the events of the view. For this, we go to the form code.

In the same way as before, in the associate and raise view events method, we will invoke these events when the corresponding buttons are clicked.

Let's start with the add new event. So, we'll invoke the add new event when the add new button is pressed.

We do the same for the edit, delete, save changes, and cancel events.

![[Pasted image 20240309230352.png]]


*PetView.cs*
   
    public partial class PetView : Form, IPetView
    
    {
        //Fields
        private string message;
        private bool isSuccessful;
        private bool isEdit;

        //Constructor
        public PetView()
        {
            InitializeComponent();
            AssociateAndRaiseViewEvents();
            tabControl1.TabPages.Remove(tabPagePetDetail);
            btnClose.Click += delegate { this.Close(); };
        }

        private void AssociateAndRaiseViewEvents()
        {
            //Search
            btnSearch.Click += delegate { SearchEvent?.Invoke(this, EventArgs.Empty); };
            txtSearch.KeyDown += (s, e) =>
              {
                  if (e.KeyCode == Keys.Enter)
                      SearchEvent?.Invoke(this, EventArgs.Empty);
              };
            //Add new
            btnAddNew.Click += delegate
            {
                AddNewEvent?.Invoke(this, EventArgs.Empty);
                tabControl1.TabPages.Remove(tabPagePetList);
                tabControl1.TabPages.Add(tabPagePetDetail);
                tabPagePetDetail.Text = "Add new pet";
            };
            //Edit
            btnEdit.Click += delegate
            {
                EditEvent?.Invoke(this, EventArgs.Empty);
                tabControl1.TabPages.Remove(tabPagePetList);
                tabControl1.TabPages.Add(tabPagePetDetail);
                tabPagePetDetail.Text = "Edit pet";
            };
            //Save changes
            btnSave.Click += delegate
            {
                SaveEvent?.Invoke(this, EventArgs.Empty);
                if (isSuccessful)
                {
                    tabControl1.TabPages.Remove(tabPagePetDetail);
                    tabControl1.TabPages.Add(tabPagePetList);
                }
                MessageBox.Show(Message);
            };
            //Cancel
            btnCancel.Click += delegate
            {
                CancelEvent?.Invoke(this, EventArgs.Empty);
                tabControl1.TabPages.Remove(tabPagePetDetail);
                tabControl1.TabPages.Add(tabPagePetList);
            };
            //Delete
            btnDelete.Click += delegate
            {               
                var result = MessageBox.Show("Are you sure you want to delete the selected pet?", "Warning",
                      MessageBoxButtons.YesNo, MessageBoxIcon.Warning);
                if (result == DialogResult.Yes)
                {
                    DeleteEvent?.Invoke(this, EventArgs.Empty);
                    MessageBox.Show(Message);
                }
            };
        }


Well, once we have finished associating the events, we must show the detail tab and hide the data list tab when the add or edit button is clicked. In the same way, when the changes are saved correctly or cancel is clicked, we must hide the detail tab and show the data list again.

So, when the click event of the add button is raised, we remove the pet list tab and add the pet detail tab. To make the app a bit more interactive, we'll also change the title of the tab, for example, "Add New Pet".

We do the same for when the edit event is raised. Now, when the save button is clicked, the detail tab should be hidden, and the pet list shown again. However, that should happen as long as the operation has been successful. The value of this property will be set from the presenter.

So, if the operation was successful, we remove the detail tab and re-add the pet list tab. We will also display a message box with the result of the operation.

In the same way, for the cancel event, the detail tab must be hidden, and the list of pets must be shown again.

Finally, in the click event of the delete button, in this case, the pet will be deleted directly without going through the detail tab. Then we will show a warning message with yes and no buttons, asking if the user is sure to delete the selected pet.

The show method of the message box returns a dialog result, so we assign it to a variable and check if the user clicked the yes button. If true, we invoke the delete event.

Similarly, as in the save event, we also display the result message of the operation.

All right, that's all for the view. Now, we'll implement the event handler methods, that is, what the app should do when the add, edit, or delete event is raised.

To do this, we go to the pet presenter class. In this first part of the tutorial, we subscribe to the events of the view. For example, the method "load data of the selected pet to be edited" is subscribed to the edit event.

Now, we will implement these event handler methods, that is, what is going to be done when the event is executed, as I mentioned above.

So, when the add new event is raised, we will only change the edit state of the view to false, since the save event will be in charge of adding or editing the data.

Now, when the edit event is raised, we will load the data of the selected pet in the text boxes of the form to be able to edit it. To do this, we need to get the pet model. So, we access the binding source of the pet list and get the object of the currently selected row. This property gets the current item from the underlying list. As I said in the first part of the tutorial, the binding source class is native to the Windows Forms assembly and has many benefits, as it creates a binding between a data control and the data source.

In the implementation of the method in the form, the binding source of the pet list is set to the data source of the DataGridView. So, all further interaction with the data is done with calls to the binding source component. So, it serves many purposes.

Well, let's continue.

Once the model to be edited is obtained, we assign the value of the properties to the properties of the view, that is, to the text boxes. Finally, we change the edit state of the view to true. This value changes based on user action, whether data is added or edited, with the save event taking care of the final work.

![[Pasted image 20240309231004.png]]

So, we implement this method.

First, we create a pet model instance. We then pass the data from the view to the model.

Okay, now we define a try-catch block to catch the errors.

If any errors occur, we tell the view that the operation was unsuccessful and also set the error message.

Before adding or editing a pet, we need to validate that the data in the model is correct. Since in the model, we did some validations, for example, not allowing empty data and setting minimum and maximum length. However, these validator attributes need to be processed to perform the corresponding validations. For that job, there are also some classes in the model components DataAnnotation assembly to do that task. 


![[Pasted image 20240309231153.png]]
So, we will create a class to validate any model. Since in an application, we will have many more models. So, in the presenter, we will add a folder for the common task classes. In it, we add a class to validate the model data.

We define a method to validate an object. We define a field for the validation error message. We then define a list for the result of the validations and a context for the model validation.

Now, we determine if the model is valid or not. For this, we use the Validator helper class and call the "TryValidateObject" method. As parameters, we send the object, the context, the collection of results, and indicate that all properties must be validated.

Well, we define a condition. If the model is not valid, we loop through the result list to extract the validation error messages and set it to the local error message field.

Here, we can return the boolean value or message as a result of the method, and thus check it from the presenter, or we can simply throw an exception with the error messages and catch it in the try-catch block.

To make it clearer for those people who are just starting out in this, in case the data of the model is not valid, this exception will be caught by the try-catch block in the presenter, and the exception message is set to the view's result message.

The data validation error message is assigned in the model validation attributes. Therefore, these messages are the ones that will be displayed in the view in case the values of the model properties are not valid. All right, let's continue.

public class ModelDataValidation
{
    public void Validate(object model)
    {
        string errorMessage = "";
        List<ValidationResult> results = new List<ValidationResult>();
        ValidationContext context = new ValidationContext(model);
        bool isValid = Validator.TryValidateObject(model,context,results,true);
        if(isValid==false)
        {
            foreach (var item in results)
                errorMessage += "- " + item.ErrorMessage + "\n";
            throw new Exception(errorMessage);
        }
    }
}



So, before adding or editing, we validate the pet model.

*PetPresenter.cs*

public class PetPresenter
{
    //Fields
    private IPetView view;
    private IPetRepository repository;
    private BindingSource petsBindingSource;
    private IEnumerable<PetModel> petList;

    //Constructor
    public PetPresenter(IPetView view, IPetRepository repository)
    {
        this.petsBindingSource = new BindingSource();
        this.view = view;
        this.repository = repository;
        //Subscribe event handler methods to view events
        this.view.SearchEvent += SearchPet;
        this.view.AddNewEvent += AddNewPet;
        this.view.EditEvent += LoadSelectedPetToEdit;
        this.view.DeleteEvent += DeleteSelectedPet;
        this.view.SaveEvent += SavePet;
        this.view.CancelEvent += CancelAction;
        //Set pets bindind source
        this.view.SetPetListBindingSource(petsBindingSource);
        //Load pet list view
        LoadAllPetList();
        //Show view
        this.view.Show();
    }

    //Methods
    private void LoadAllPetList()
    {
        petList = repository.GetAll();
        petsBindingSource.DataSource = petList;//Set data source.
    }
    private void SearchPet(object sender, EventArgs e)
    {
        bool emptyValue = string.IsNullOrWhiteSpace(this.view.SearchValue);
        if (emptyValue == false)
            petList = repository.GetByValue(this.view.SearchValue);
        else petList = repository.GetAll();
        petsBindingSource.DataSource = petList;
    }
    private void AddNewPet(object sender, EventArgs e)
    {
        view.IsEdit = false;          
    }
    private void LoadSelectedPetToEdit(object sender, EventArgs e)
    {
        var pet = (PetModel)petsBindingSource.Current;
        view.PetId = pet.Id.ToString();
        view.PetName = pet.Name;
        view.PetType = pet.Type;
        view.PetColour = pet.Colour;
        view.IsEdit = true;
    }
    private void SavePet(object sender, EventArgs e)
    {
        var model = new PetModel();
        model.Id = Convert.ToInt32(view.PetId);
        model.Name = view.PetName;
        model.Type = view.PetType;
        model.Colour = view.PetColour;
        try
        {
            new Common.ModelDataValidation().Validate(model);
            if(view.IsEdit)//Edit model
            {
                repository.Edit(model);
                view.Message = "Pet edited successfuly";
            }
            else //Add new model
            {
                repository.Add(model);
                view.Message = "Pet added sucessfully";
            }
            view.IsSuccessful = true;
            LoadAllPetList();
            CleanviewFields();
        }
        catch (Exception ex)
        {
            view.IsSuccessful = false;
            view.Message = ex.Message;
        }
    }

    private void CleanviewFields()
    {
        view.PetId = "0";
        view.PetName = "";
        view.PetType = "";
        view.PetColour = "";            
    }

    private void CancelAction(object sender, EventArgs e)
    {
        CleanviewFields();
    }
    private void DeleteSelectedPet(object sender, EventArgs e)
    {
        try
        {
            var pet = (PetModel)petsBindingSource.Current;
            repository.Delete(pet.Id);
            view.IsSuccessful = true;
            view.Message = "Pet deleted successfully";
            LoadAllPetList();
        }
        catch (Exception ex)
        {
            view.IsSuccessful = false;
            view.Message = "An error ocurred, could not delete pet";
        }
    }

}

Now, if the edit state of the view is true, we edit the model. For this, we call the edit method of the repository and as a parameter, we send the model. Then, we set the result message, for example, "Pet successfully edited." Otherwise, if the condition is not met, we add a new pet record. Finally, we establish that the operation was successful, and we refresh the view using the "load all pet list" method. Also, after saving the changes, we will clean the text boxes of the view.

The default value of the ID field should be zero, as this is a numeric field. Otherwise, an error would be thrown. In the rest, we set empty strings.

All right, in the cancel event, we just clear the view's fields.

Finally, we implement the functionality of the "remove selected pet" event. In the same way as above, we use a try-catch block. We get the currently selected pet model in the DataGridView's binding source. We then call the remove repository method and send the pet ID to remove. We indicate that the operation was successful and we set a message. Finally, we update the list. Now, if an error occurs, we indicate that the operation was not successful, and we set a message. It is not recommended to show all the details of the error message to the end-user but to set a short and personalized message.

All right, that's it on the presenter. Now, we only need to implement the add, edit, and delete methods of the repository.

Let's start with the add new record method. In the same way as we did in the "get all" method, using the using statement, we create the SQL connection and command object. We open the connection and set the connection to the command. Then, we assign the text command. In this case, "insert in the pets table the following values: name, type, and color." The ID is auto-incremental, so it is not necessary to insert it. Okay, now we create and add those parameters to the command's parameter collection, specify the name and data type, then set the corresponding value. Finally, we execute the operation.

We do similarly for the delete and edit method.

All right, that's it. Let's test the app.

The edit and delete function are working correctly. Data validation is also working fine.

Well, the add function is also fine. Here I forgot to set the ID text box to read-only. You can change the value, and this will produce an error, as it is not possible to convert a string to a number. Also, the ID field is auto-incrementing, so you don't need to specify this data. So, we set the number 0 in the text box as the default value and specify that the field is read-only. Although it would be better to initialize the field from the presenter, but well, for a quick fix, it's fine.

Well, everything is working fine, but this project can be improved much more. You can give it your personal touches, like improving the structure of the code and customizing the appearance.

* PetRepository.cs*

public class PetRepository : BaseRepository, IPetRepository
{
    //Constructor
    public PetRepository(string connectionString)
    {
        this.connectionString = connectionString;
    }
    //Methods
    public void Add(PetModel petModel)
    {
        using (var connection = new SqlConnection(connectionString))
        using (var command = new SqlCommand())
        {
            connection.Open();
            command.Connection = connection;
            command.CommandText = "insert into Pet values (@name, @type, @colour)";
            command.Parameters.Add("@name", SqlDbType.NVarChar).Value = petModel.Name;
            command.Parameters.Add("@type", SqlDbType.NVarChar).Value = petModel.Type;
            command.Parameters.Add("@colour", SqlDbType.NVarChar).Value = petModel.Colour;
            command.ExecuteNonQuery();
        }
    }
    public void Delete(int id)
    {
        using (var connection = new SqlConnection(connectionString))
        using (var command = new SqlCommand())
        {
            connection.Open();
            command.Connection = connection;
            command.CommandText = "delete from Pet where Pet_Id=@id";
            command.Parameters.Add("@id", SqlDbType.Int).Value = id;           
            command.ExecuteNonQuery();
        }
    }
    public void Edit(PetModel petModel)
    {
        using (var connection = new SqlConnection(connectionString))
        using (var command = new SqlCommand())
        {
            connection.Open();
            command.Connection = connection;
            command.CommandText = @"update Pet 
                                    set Pet_Name=@name,Pet_Type= @type,Pet_Colour= @colour 
                                    where Pet_Id=@id";
            command.Parameters.Add("@name", SqlDbType.NVarChar).Value = petModel.Name;
            command.Parameters.Add("@type", SqlDbType.NVarChar).Value = petModel.Type;
            command.Parameters.Add("@colour", SqlDbType.NVarChar).Value = petModel.Colour;
            command.Parameters.Add("@id", SqlDbType.Int).Value = petModel.Id;
            command.ExecuteNonQuery();
        }
    }