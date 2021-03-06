# PPAW

## _**Lab 1**_
<details unu>
  <summary>Apasati pentru a extinde laboratorul 1!</summary>

### **Exercitii:**
#### _*1. Adaugati un nou camp clasei Masina din cadrul proiectului demo si modificati toate nivelurile aplicatiei WinForms sa reflecte aceasta schimbare.*_

_*Campul adaugat este **Culoare** de tip string.*_
  
**a. Masina.cs**
```csharp
namespace LibrarieModele{
    public class Masina {
      // ...
      public string Culoare {get; set;}
  
      public Masina(DateTime dataFabricatie, int idCompanie, string model, decimal pret, string culoare, int idMasina = 0)
      {
        // ...
        Culoare = culoare;
      }
  
      public Masina (DataRow linieBD)
      {
          // ...
          Culoare = linieBD["culoare"].ToString();
      }
    } // class Masina
  } // namespace Librarie Modele
```

**b. AdministrareMasini.cs**
```csharp  
  namespace NivelAccesDate_SQLServer
{
    public class AdministrareMasini: IStocareMasini
    {
       public bool AddMasina(Masina m)
        {
            return SqlDBHelper.ExecuteNonQuery(
                "insert into dbo.Masini VALUES (@DataFabricatie, @IdCompanie, @Model, @Pret, @Culoare)", CommandType.Text,
                // ...
                new SqlParameter("@Culoare", m.Culoare)
            );
        }

        public bool UpdateMasina(Masina m)
        {
            return SqlDBHelper.ExecuteNonQuery(
                "UPDATE dbo.Masini set dataFabricatie = @DataFabricatie, idCompanie = @IdCompanie, model =@Model, pret =@Pret, culoare = @Culoare where idMasina=@IdMasina", CommandType.Text,
                // ...
                new SqlParameter("@Culoare", m.Culoare)
            );
        }
    } // class AdministrareMasini
} // namespace NivelAccesDate_SQLServer  
```
  
**c. FormaAdaugare.cs**
```csharp
namespace InterfataUtilizator
{
    public partial class FormaAdaugare : MetroForm
    {
      // ...
       private void dataGridMasini_CellContentClick(object sender, DataGridViewCellEventArgs e)
        {
            this.Width = 1000;
            int currentRowIndex = dataGridMasini.CurrentCell.RowIndex;
            string idMasina = dataGridMasini[PRIMA_COLOANA, currentRowIndex].Value.ToString();

            try
            {
                Masina m = stocareMasini.GetMasina(Int32.Parse(idMasina));

                //incarcarea datelor in controalele de pe forma
                if (m != null)
                {
                    // ...
                    txtCuloare.Text = m.Culoare; // adaugarea campului Culoare
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message.ToString());
            }
            pnlEditare.Visible = true;
        }

        private void btnActualizeaza_Click(object sender, EventArgs e)
        {
            try
            {
                var masina = new Masina(
                    dtDataFabricatie.Value,
                    ((ComboItem)cmbCompanii.SelectedItem).Value,
                    txtModel.Text,
                    decimal.Parse(txtPret.Text),
                    txtCuloare.Text, // adaugarea metroBox-ului Culoare
                    Int32.Parse(lblIdMasina.Text));

                var rezultat = stocareMasini.UpdateMasina(masina);
                if (rezultat == SUCCES)
                {
                    MessageBox.Show("Masina actualizata");
                    AfiseazaCatalog();
                }
                else
                {
                    MessageBox.Show("Eroare la actualizare masina");
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show("Exceptie" + ex.Message);
            }
        }

      // ...      
    } // class FormaAdaugare
} // namespace InterfataUtilizator
```
  
**d. FormaAfisare**
  
```csharp
namespace InterfataUtilizator
{
    public partial class FormaAfisare : MetroForm
    {
       // ...
    
        private void AfiseazaCatalog()
        {
            try
            {
                var masini = stocareMasini.GetMasini();
                if (masini != null && masini.Any())
                { // adugare campului "Culoare" in Select
                    dataGridMasini.DataSource = masini.Select(m=> new { m.IdMasina, m.Model, m.Companie.Nume, m.DataFabricatie, m.Pret, m.Culoare }).ToList() ;

                    dataGridMasini.Columns[0].Visible = false;
                    dataGridMasini.Columns[2].HeaderText = "Companie";
                    dataGridMasini.Columns[3].HeaderText = "Data fabricatie";
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message.ToString());
            }
        }
    
    } // class FormaAfisare
 } // namespace InterfataUtilizator
  
```

### *Rezultate:*
  
**1. Afisare si modificare date masina**
  
  ![afisare_modoficare_masini](https://user-images.githubusercontent.com/39569343/139529075-649f81d6-4473-457f-a57a-52e077e768f7.png)
  
**2. Adaugare masina**  
  
  ![adaugare_masina](https://user-images.githubusercontent.com/39569343/139529176-c3b16af7-296c-40ec-9762-44aca0298b84.png)
</details>

## _**Lab 2**_
<details>
  <summary>Apasati pentru a deschide laboratorul 2!</summary>
  
  ### _**Exercitii:**_
  #### *2. Adaugati o noua forma care sa afiseze toate companiile introduse.*
  
  **a. ListaCompanii.aspx**
  ```csharp
  <%@ Page Title="" Language="C#" MasterPageFile="~/Site.Master" AutoEventWireup="true" CodeBehind="ListaCompanii.aspx.cs" Inherits="InterfataUtilizator_WebForms.ListaCompanii" %>
<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <section>
        <div>
            <hgroup>
                <h2><%: Page.Title %></h2>
            </hgroup>

            <asp:ListView ID="productList" runat="server" 
                DataKeyNames="IdCompanie" GroupItemCount="4"
                ItemType="LibrarieModele.Companie" SelectMethod="GetCompanii">
                <EmptyDataTemplate>
                    <table >
                        <tr>
                            <td>Nu a fost gasita nici o companie.</td>
                        </tr>
                    </table>
                </EmptyDataTemplate>
                <EmptyItemTemplate>
                    <td/>
                </EmptyItemTemplate>
                <GroupTemplate>
                    <tr id="itemPlaceholderContainer" runat="server">
                        <td id="itemPlaceholder" runat="server"></td>
                    </tr>
                </GroupTemplate>
                <ItemTemplate>
                    <td runat="server">
                        <table>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                        <img src="./Resources/Images/<%#:Item.Nume%>.jpg"
                                            width="100" height="75" style="border: solid" /></a>
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                        <span>
                                            <%#:Item.Nume%>
                                        </span>
                                    </a>
                                    <br />
                                    <span>
                                        <b>Adresa: </b><%#:Item.Adresa%>
                                    </span>
                                    <br />
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                    </a>
                                    <br />
                                    <span>
                                        <b>Email: </b><%#:Item.Email%>
                                    </span>
                                    <br />
                                </td>
                            </tr>
                            <tr>
                                <td>
                                    <a href="ProductDetails.aspx?productID=<%#:Item.IdCompanie%>">
                                    </a>
                                    <br />
                                    <span>
                                        <b>Telefon: </b><%#:Item.Telefon%>
                                    </span>
                                    <br />
                                </td>
                            </tr>
                            <tr>
                                <td>&nbsp;</td>
                            </tr>
                        </table>
                        </p>
                    </td>
                </ItemTemplate>
                <LayoutTemplate>
                    <table style="width:100%;">
                        <tbody>
                            <tr>
                                <td>
                                    <table id="groupPlaceholderContainer" runat="server" style="width:100%">
                                        <tr id="groupPlaceholder"></tr>
                                    </table>
                                </td>
                            </tr>
                            <tr>
                                <td></td>
                            </tr>
                            <tr></tr>
                        </tbody>
                    </table>
                </LayoutTemplate>
            </asp:ListView>
        </div>
    </section>
</asp:Content>
 ```
**b. Lista Companii.aspx.cs**
  ```csharp
  namespace InterfataUtilizator_WebForms
{
    public partial class ListaCompanii : System.Web.UI.Page
    {
        //initializare obiecte utilizate pentru salvarea datelor in baza de date (sau alte medii de stocare...daca exista implementare corespunzatoare)
        IStocareCompanii stocareCompanii = (IStocareCompanii)new StocareFactory().GetTipStocare(typeof(Companie));
        IStocareMasini stocareMasini = (IStocareMasini)new StocareFactory().GetTipStocare(typeof(Masina));

        protected void Page_Load(object sender, EventArgs e)
        {

        }
        public IQueryable<Companie> GetCompanii()
        {
            return stocareCompanii.GetCompanii().AsQueryable();
        }
    } // parial class ListaCompanii
} // namespace InterfataUtilizator_WebForms
  ```
**c. ListaCompanii.aspx.designer.cs**
  ```csharp
  namespace InterfataUtilizator_WebForms
{
    public partial class ListaCompanii
    {
        protected global::System.Web.UI.WebControls.ListView productList;
    } 
}
  ```
  
  **d. Site.Master**
  ```html
  <div class="navbar-collapse collapse">
      <ul class="nav navbar-nav">
          <li><a runat="server" href="~/">Home</a></li>
          <li><a runat="server" href="~/ListaMasini">Lista masini</a></li>
          <li><a runat="server" href="~/ListaCompanii">Lista companii</a></li> <!--Adaugare tab lista companii-->
          <li><a runat="server" href="~/AdaugareCompanie">AdaugareCompanie</a></li>
      </ul>
  </div>
  ```
  
  ### *Rezultate:*
  
  ![SiteMaster](https://user-images.githubusercontent.com/39569343/139533821-cb855d9b-a85b-41cc-a8b6-6a7337ee6749.png)
  
</details>

## _**Lab 3**_
<details>
  <summary>Apasti pentru a deschide laboratorul 3!</summary>
  
  ### _**Exercitii:**_
  #### _*1. Implementati operatiile de Index si Details pentru entitatea considerata in cadrul proiectului propriu utilizand paradigma MVC.*_
  ##### 1.1 Proiect web de tip mvc - https://github.com/bmarian98/java_mvc

##### Referinte catre pachete "ro.ppaw.dao" = NivelAccesDate, "ro.ppaw.beans" = Librarie Model.
	
Referinta obiectului petDao din clasa PetDao.jva pachetul "ro.ppaw.dao" este trimis prin Dependancy Injection	
	
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd">  

<context:component-scan base-package="ro.ppaw.controllers"></context:component-scan>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp/"></property>
	<property name="suffix" value=".jsp"></property>
</bean>

<bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>  
	<property name="url" value="jdbc:mysql://localhost:3306/mvc"></property>  
	<property name="username" value="root"></property>  
	<property name="password" value="new123"></property>  
</bean>  

<bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate">
<property name="dataSource" ref="ds"></property>
</bean>


<bean id="petDao" class="ro.ppaw.dao.PetDao">
	<property name="template" ref="jt"></property>
</bean>	
```	
	
  Modelul Shelter.java
  ```java
public class Shelter {

	@Override
	public String toString() {
		return "Shelter [id=" + id + ", name=" + name + ", address=" + address + "]";
	}

	private Integer id;
	private String name;
	private String address;

	public Shelter() {
	}

	public Shelter(Integer id, String name, String address) {
		super();
		this.id = id;
		this.name = name;
		this.address = address;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public Integer getId() {
		return id;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setAddress(String address) {
		this.address = address;
	}

	public String getAddress() {
		return address;
	}
}
  ```
  ##### 1.2 Controller actiunile: Index si Details din Controller
  ShelterController.java
  ```java
  // ...
  // index
 @RequestMapping("/list_shelters")
	public String list(Model m) {
		List<Shelter> list = shelterDao.getShelters();
		m.addAttribute("list", list);
		return "list_shelters";
	}
  
  //details
  @RequestMapping(value = "/shelter_details/{id}")
	public String datails(@PathVariable Integer id, Model m) {
		Shelter shelter = shelterDao.getShelter(id);
		m.addAttribute("command", shelter);
		return "shelter_details";
	}
  ```
  
  ##### 1.3 Adaugare referinta modele si acces date
  spring-servlet.xml
  ```xml
  <beans>
    <bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
      <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>  
      <property name="url" value="jdbc:mysql://localhost:3306/mvc"></property>  
      <property name="username" value="root"></property>  
      <property name="password" value="new123"></property>  
    </bean>  

    <bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate">
     <property name="dataSource" ref="ds"></property>
    </bean>

    <bean id="shelterDao" class="com.javatpoint.dao.ShelterDao">
      <property name="template" ref="jt"></property>
    </bean>
  </beans>
  ```
  
  ##### 1.4 View-uri
  a. list_shelters.jsp
  ```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Shelters List</h1>
<table border="2" width="70%" cellpadding="2">
	<tr><th>Name</th><th>Details</th><th>Edit</th></tr>
	<c:forEach var="shelter" items="${list}"> 
		<tr>
			<td>${shelter.name}</td>
			<td><a href="shelter_details/${shelter.id}">Details</a></td>
			<td><a href="edit_shelter/${shelter.id}">Edit</a></td>
		</tr>
	</c:forEach>
</table>
<br/>
<a href="shelterform">Add New Shelter</a>
  ```
  b. shelter_detail.jsp
  
  ```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Shelter details</h1>
<table  border="2" width="70%" cellpadding="2">
	<tr><th>Id</th><th>Name</th><th>Address</th></tr>
	<tr>
	    <td><c:out value="${command.id}" /></td>
	    <td><c:out value="${command.name}" /></td>
	    <td><c:out value="${command.address}" /></td>  
      </tr>  
</table>  
<a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>    
  ```	    
  
  ##### 1.5 Testare
  a. List view page
  ![shelter_list1](https://user-images.githubusercontent.com/39569343/142162552-27a15172-47e1-4ddd-b650-fbfe3ffa38ab.png)
      
  b. Details view page	
  ![shelter_details](https://user-images.githubusercontent.com/39569343/142162550-d856542c-0e96-4d42-8aa3-841cdbda3347.png)
      
 #### _*2. Implementati operatiile de Create si Edit pentru entitatea considerata in cadrul proiectului propriu utilizand paradigma MVC.*_
 ##### Metodele create si save din Controler
 ShelterController.java
 ```java
@Controller
public class ShelterController {

	@Autowired
	ShelterDao shelterDao;

	@RequestMapping("/shelterform")
	public String showform(Model m) {
		m.addAttribute("command", new Shelter());
		return "shelterform";
	}

	// insereaza datele in baza de date
	@RequestMapping(value = "/save_shelter", method = RequestMethod.POST)
	public String save(@ModelAttribute("shelter") Shelter shelter) {
		System.out.println(shelter);
		shelterDao.save(shelter);
		return "redirect:/list_shelters";
	}
	
	// extrage obiectul dupa id din baza de date si permite editarea acestuia
	@RequestMapping(value = "/edit_shelter/{id}")
	public String edit(@PathVariable Integer id, Model m) {
		Shelter shelter = shelterDao.getShelter(id);
		m.addAttribute("command", shelter);

		return "edit_shelter";
	}

	// updateaza obiectul si il salveaza in baza de date
	@RequestMapping(value = "/edit_save_shelter", method = RequestMethod.POST)
	public String editsave(@ModelAttribute("shelter") Shelter shelter) {
		shelterDao.update(shelter);

		return "redirect:/list_shelters";
	}

}
 ```
##### View-uri pentru adaugare si editare
a. shelterform.jsp
```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>    
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>    
    
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Add new Shelter</title>
</head>
<body>
	<h1>Add shelter</h1>
	
	<form:form method="post" action="save_shelter">
		<table>
			<tr>
				<td>Id:</td>
				<td><form:input path="id" /></td>
			</tr>
			<tr>
				<td>Name:</td>
				<td><form:input path="name" /></td>
			</tr>
			<tr>
				<td>Address:</td>
				<td><form:input path="address" /></td>
			</tr>
			<tr>
				<td></td>
				<td><input type="submit" value="Save" /></td>
			</tr>
		</table>
	</form:form>
	<a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>
</body>
</html>	
```
b. edit_helter.java
```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Edit shelter</h1>
<form:form method="POST" action="/SpringMVCCRUDSimple/edit_save_shelter">  
<table >  
	<tr>
	<td></td>  
	 	<td><form:hidden  path="id" /></td>
	 </tr> 
	 <tr>  
		  <td>Name : </td> 
		  <td><form:input path="name"  /></td>
	 </tr>  
	 <tr>  
		  <td>Address :</td>  
		  <td><form:input path="address" /></td>
	 </tr> 
	 <tr>  
		  <td> </td>  
		  <td><input type="submit" value="Edit Save" /></td>  
	 </tr>  
</table>  
</form:form>  
```
##### Rezultate
a. Adugare
	
![add_shelter](https://user-images.githubusercontent.com/39569343/142162540-efb94a21-90b6-4e3a-8e5d-8e50a2ba50b0.png)
	
b. Editare
	
![edit_shelter](https://user-images.githubusercontent.com/39569343/142162543-c65bebba-722c-41a2-aa16-b420a896b465.png)
![list_shelter2](https://user-images.githubusercontent.com/39569343/142162547-23075bdd-d6f0-4ce0-a6b7-3f9e56bf8319.png)
  
</details>

## _**Lab 4**_
<details>
  <summary>Apasti pentru a deschide laboratorul 4!</summary>
	
### _**Exercitii:**_
	
#### *1. Implementati operatiile de Index si Create pentru o a doua entitate considerata in cadrul proiectului propriu. Aceasta entitate trebuie sa contina o cheie straina. In acest fel, modelul implementat va contine date din mai multe tabele (In cadrul exemplului de la curs am considerat MasinaModel).*
	
##### Referinte catre pachete "ro.ppaw.dao" = NivelAccesDate, "ro.ppaw.beans" = Librarie Model.
	
Referinta obiectului petDao din clasa PetDao.jva pachetul "ro.ppaw.dao" este trimis prin Dependancy Injection	
	
Referinta obiectului shelterDao din clasa ShelterDao.jva pachetul "ro.ppaw.dao" este trimis prin Dependancy Injection	
	
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:p="http://www.springframework.org/schema/p"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-3.0.xsd">  

<context:component-scan base-package="ro.ppaw.controllers"></context:component-scan>

<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	<property name="prefix" value="/WEB-INF/jsp/"></property>
	<property name="suffix" value=".jsp"></property>
</bean>

<bean id="ds" class="org.springframework.jdbc.datasource.DriverManagerDataSource">  
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>  
	<property name="url" value="jdbc:mysql://localhost:3306/mvc"></property>  
	<property name="username" value="root"></property>  
	<property name="password" value="new123"></property>  
</bean>  

<bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate">
<property name="dataSource" ref="ds"></property>
</bean>


<bean id="petDao" class="ro.ppaw.dao.PetDao">
	<property name="template" ref="jt"></property>
</bean>	
	
<bean id="shelterDao" class="ro.ppaw.dao.ShelterDao">
	<property name="template" ref="jt"></property>
</bean>
```
	
##### Modelul Pet.java
```java
public class Pet {
	private Integer id;
	private Integer shelterId;
	private String name;
	private String dateBirth;
	private Character sex;

	public Pet() {
	}

	public Pet(Integer id, Integer shelterId, String name, String dateBirth, Character sex) {
		super();
		this.id = id;
		this.shelterId = shelterId;
		this.name = name;
		this.dateBirth = dateBirth;
		this.sex = sex;
	}

	public Integer getId() {
		return id;
	}

	public void setId(Integer id) {
		this.id = id;
	}

	public Integer getShelterId() {
		return shelterId;
	}

	public void setShelterId(Integer shelterId) {
		this.shelterId = shelterId;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getDateBirth() {
		return dateBirth;
	}

	public void setDateBirth(String dateBirth) {
		this.dateBirth = dateBirth;
	}

	public Character getSex() {
		return sex;
	}

	public void setSex(Character sex) {
		this.sex = sex;
	}

	@Override
	public String toString() {
		return "Pet [shelterId=" + shelterId + ", name=" + name + ", dateBirth=" + dateBirth + ", sex=" + sex + "]";
	}

}
```
##### Controller-ul PetControler.java
```java
@Controller()
public class PetController {

	@Autowired
	PetDao petDao;

	@RequestMapping("/petform")
	public String showform(Model m) {
		m.addAttribute("command", new Pet());
		return "petform";
	}

	@RequestMapping(value = "/save_pet", method = RequestMethod.POST)
	public String save(@ModelAttribute("pet") Pet pet) {
		petDao.save(pet);
		return "redirect:/list_pets";
	}

	@RequestMapping("/list_pets")
	public String list(Model m) {
		List<Pet> list = petDao.getPets();
		m.addAttribute("list", list);
		return "list_pets";
	}

	@RequestMapping(value = "/pet_details/{id}")
	public String detilas(@PathVariable int id, Model m) {
		Pet pet = petDao.getPet(id);
		m.addAttribute("command", pet);
		return "pet_details";
	}

	@RequestMapping(value = "/edit_tmp_pet/{id}")
	public String edit_tmp_pet(@PathVariable int id, Model m) {
		Pet pet = petDao.getPet(id);
		m.addAttribute("command", pet);
		return "pet_edit_form";
	}

	@RequestMapping(value = "/edit_save_pet", method = RequestMethod.POST)
	public String editsave(@ModelAttribute("pet") Pet pet) {
		petDao.update(pet);

		return "redirect:/list_pets";
	}
}		
```
##### View-uri pentru list, details, create si edit
a. Add - petform.jsp
```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>    
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>    
<%@ page import="java.sql.*, java.io.*"%> 
    
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Add new Pet</title>
</head>
<body>
	<h1>Add pet</h1>
	
	<form:form method="post" action="save_pet">
		<table>
			<tr>
				<td>Name:</td>
				<td><form:input path="name" /></td>
			</tr>
			         <tr>
          <td>Shelter :</td>  
          <td>
          <select id="sel" onchange="fun()">
			<%
			
			try
			{
				String query = "SELECT * FROM shelter";
	        	  
	        	  Class.forName("com.mysql.jdbc.Driver");
	        	  Connection con=DriverManager.getConnection("jdbc:mysql://localhost:3306/mvc?useSSL=false", "root", "new123");
	        	  Statement st = con.createStatement();
	        	  
	        	  ResultSet rs = st.executeQuery(query);
	        	  
	        	  %> 
	        	  <option selected="selected" value="-">------</option> 
	        	  <%
	        			  
	        	  while(rs.next())
	        	  {
	        		  %>
	        		  	<option value="<%=rs.getInt(1)%>"><%=rs.getString(2) %></option>
	        		  <%
	        	  }
	        	  
			}
			catch(Exception e)
			{
				
			}
			
			%>
          </select>
          </td>
         </tr> 
			<tr>
				<td><form:input path="shelterId" id="sid" type="hidden" /></td>
			</tr>
			<tr>
				<td>DateBirth:</td>
				<td><form:input path="dateBirth" /></td>
			</tr>
			<tr>
				<td>PetSex:</td>
				<td><form:radiobutton path="sex" value="M" />Mascul</td>
				<td><form:radiobutton path="sex" value="F" />Femela</td>
			</tr>
			<tr>
				<td></td>
				<td><input type="submit" value="Save" /></td>
			</tr>
		</table>
	</form:form>
	 <script >
       function fun(){
       	console.log("test");
       	var e = document.getElementById("sel");
       	console.log(e.value);
       	document.getElementById("sid").value = e.value;
       	var d = document.getElementById("sid");
       	console.log(d.value);
       }
       </script>
	
<a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>
</body>
</html>
```
	
b.  List - list_pets.jsp
```jsp
    <%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
    <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

	<h1>Pets List</h1>
	<table  border="2" width="70%" cellpadding="2">
	<tr><th>Pet Id</th><th>Details</th><th>Edit</th></tr>
    <c:forEach var="pet" items="${list}"> 
    <tr>
    <td>${pet.name}</td>
    <td><a href="pet_details/${pet.id}">Details</a></td>
    <td><a href="edit_tmp_pet/${pet.id}">Edit</a></td>
    </tr>
    </c:forEach>
    </table>
    <br/>
    <a href="petform">Add New Pet</a>
    <a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>	
```

c. Details - pet_details.java
```java
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  

<h1>Pet details</h1>
	<table border="2" width="70%" cellpadding="2">
		<tr><th>Id</th><th>Name</th><th>Shelter_id</th><th>Date birth</th><th>Sex</th></tr>
	   	<tr>
		    <td><c:out value="${command.id}" /></td>
		    <td><c:out value="${command.name}" /></td>
		    <td><c:out value="${command.shelterId}" /></td>  
		    <td><c:out value="${command.dateBirth}" /></td>
		    <td><c:out value="${command.sex}" /></td>
	      </tr>  
     </table>  
     <a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>	    
```  

d. Edit - pet_edit_form.jsp
```jsp
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>  
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>  
 <%@ page import="java.sql.*, java.io.*"%> 

		<h1>Edit pet</h1>
		 
       <form:form method="PUT" action="/SpringMVCCRUDSimple/edit_save_pet">  
      	<table >  
      	<tr>
      	<td></td>  
         <td><form:hidden  path="id" id="sid" /></td>
         </tr> 
         <tr>  
          <td>Name : </td> 
          <td><form:input path="name" /></td>
         </tr>  
         <tr>
          <td>Shelter :</td>  
          <td>
          <select id="sel" onchange="fun()">
			<%
			
			try
			{
				String query = "SELECT * FROM shelter";
	        	  
	        	  Class.forName("com.mysql.jdbc.Driver");
	        	  Connection con=DriverManager.getConnection("jdbc:mysql://localhost:3306/mvc?useSSL=false", "root", "new123");
	        	  Statement st = con.createStatement();
	        	  
	        	  ResultSet rs = st.executeQuery(query);
	        	  
	        	  %> 
	        	  <option selected="selected" value="-">------</option> 
	        	  <%
	        			  
	        	  while(rs.next())
	        	  {
	        		  %>
	        		  	<option value="<%=rs.getInt(1)%>"><%=rs.getString(2) %></option>
	        		  <%
	        	  }
	        	  
			}
			catch(Exception e)
			{
				
			}
			
			%>
          </select>
          </td>
         </tr> 
         <tr>  
          <td>DateBirth :</td>  
          <td><form:input path="dateBirth" name="sel" /></td>
         </tr> 
         <tr>  
          <td>Sex:</td>
				<td><form:radiobutton path="sex" value="M" />Mascul</td>
				<td><form:radiobutton path="sex" value="F" />Femela</td>
         </tr> 
         <tr>
      	<td></td>  
         <td><form:input path="shelterId" id="shelter" type="hidden" /></td>
         </tr> 
         <tr>  
          <td> </td>  
          <td><input type="submit" value="Edit Save" /></td>  
         </tr>  
        </table>  
       </form:form>  
       
       <script >
       function fun(){
       	console.log("test");
       	var e = document.getElementById("sel");
       	console.log(e.value);
       	document.getElementById("shelter").value = e.value;
       	var d = document.getElementById("shelter");
       	console.log(d.value);
       }
       </script>
      
       
       <a href="/SpringMVCCRUDSimple/index.jsp">HOME</a>
```
	
##### Rezultate
a. Add view
	
![add](https://user-images.githubusercontent.com/39569343/143256651-38701fcb-eed3-42bc-8415-1053d3391d09.png)

b. List view
	
![list_after_add](https://user-images.githubusercontent.com/39569343/143256648-45826f05-1a41-4d23-967b-18b929534495.png)

c. Edit view
	
![edit](https://user-images.githubusercontent.com/39569343/143256637-f0947066-9566-4f1a-93a0-81af2d635aa8.png)

d. Details view
	
![info](https://user-images.githubusercontent.com/39569343/143256646-f36d8157-37a0-4638-8eb2-a5e60c9da300.png)
	
</details>

## _**Lab 5**_
<details>
  <summary>Apasti pentru a deschide laboratorul 5!</summary>
	
### _**Exercitii:**_
	
#### *1. Implementa??i opera??iile de Get si Get(int id) pentru entitatea considerata in cadrul proiectului propriu utiliz??nd paradigma API.*
	
Proiect WEB API
	
https://github.com/bmarian98/pp_aw
	
Crearea tabelelor din model

Pet.java
```java
@Entity
public class Pet implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;
    private String name;
    private String species;
    private String dateBirth;
    private Character sex;
    private String imageUrl;

    public Pet() {}

    public Pet(Long id, String name, String species, String dateBirth, String imageUrl, Character sex) {
        this.id = id;
        this.name = name;
        this.species = species;
        this.dateBirth = dateBirth;
        this.imageUrl = imageUrl;
        this.sex = sex;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }
 // ...
}
```
	
Shelter.java
```java
@Entity
public class Shelter implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;

    @OneToMany(
          cascade = CascadeType.ALL,
          orphanRemoval = true
    )
    private List<Pet> pets = new ArrayList<>();
    private String name;
    private String address;

    public Shelter() {}

    public Shelter(Long id, String name, String address) {
        this.id = id;
        this.name = name;
        this.address = address;
    }
}
```

PetRepository.java - Repository
```java
@Repository
public interface PetRepository extends JpaRepository<Pet, Long> {
    void deletePetById(Long id);

    Optional<Pet> findPetById(Long id);
}
```
	
PetService.java - Service	
```java
@Service
@Transactional
public class PetService {

    private final PetRepository petRepo;

    @Autowired
    public PetService(PetRepository petRepo){
        this.petRepo = petRepo;
    }

    public Pet addPet(Pet pet){
        return petRepo.save(pet);
    }

    public List<Pet> findAllPets(){
        return petRepo.findAll();
    }

    public Pet updatePet(Pet pet){
        return petRepo.save(pet);
    }

    public void deletePet(Long id){
        petRepo.deletePetById(id);
    }

    public Pet findPetById(Long id){
        return petRepo.findPetById(id).orElseThrow(() -> new UserNotFoundException("Pet with id " + id + " not found"));
    }

}	
```
	
PetResource.java - RestController
```java
@RestController
@RequestMapping("/pet")
public class PetResource {
    private final PetService petService;

    public PetResource(PetService petService){
        this.petService = petService;
    }

    // GET pentru toate elementele din table
    @GetMapping("/all")
    public ResponseEntity<List<Pet>> getAllPets(){
        List<Pet> pets = petService.findAllPets();
        return new ResponseEntity<>(pets, HttpStatus.OK);
    }

    // GET cu ID
    @GetMapping("/find/{id}")
    public ResponseEntity<Pet> getPetById(@PathVariable("id") Long id){
        Pet pet = petService.findPetById(id);
        return new ResponseEntity<>(pet, HttpStatus.OK);
    }

}
```
	
#### Testare cu Postman

Get pentru toate elementele
![find_all](https://user-images.githubusercontent.com/39569343/143069933-c2e950bd-a838-4471-9f2e-5cafb0fcc35d.png)
	
Get pentru un element cu id specific
![find_by_id](https://user-images.githubusercontent.com/39569343/143069943-c0de19f9-30a8-494b-aaa1-ceed68ab3df8.png)
	
#### *2. Implementa??i opera??iile de Post si Put pentru entitatea considerata in cadrul proiectului propriu utiliz??nd paradigma API.*	
	
PetResource.java - RestController
```java
@RestController
@RequestMapping("/pet")
public class PetResource {
    private final PetService petService;

    public PetResource(PetService petService){
        this.petService = petService;
    }
	
    // ...

    @PostMapping("/add")
    public ResponseEntity<Pet> addPet(@RequestBody Pet pet){
        Pet newPet = petService.addPet(pet);
        return new ResponseEntity<>(newPet, HttpStatus.OK);
    }

    @PutMapping("/edit")
    public ResponseEntity<Pet> editPet(@RequestBody Pet pet){
        Pet editPet = petService.updatePet(pet);
        return new ResponseEntity<>(editPet, HttpStatus.OK);
    }
}
```

#### Testare POST si PUT 
	
![post_pet](https://user-images.githubusercontent.com/39569343/143071004-f70208f5-25ef-4aa3-933d-4216059952e1.png)

![edit_put](https://user-images.githubusercontent.com/39569343/143071039-2bdaffe8-dbb0-4ed6-9c15-632a5d2d817f.png)
	
</details>

## _**Lab 6**_
<details>
  <summary>Apasti pentru a deschide laboratorul 6!</summary>
	
### _**Exercitii:**_
	
#### *1. Crea??i pagini web in care sa utilizati operatiile CRUD API implementate in laboratorul 5.*
Link catre frontend
	
https://github.com/bmarian98/frontend_shelter
	
Realizarea paginilor s-a realizat cu framwork-ul Angular

Crearea unei interfete asemanatoare cu modelul

/app/pet/pet.ts
```ts
export interface Pet{
    id: number;
    name: string;
    species: string;
    dateBirth: string;
    imageUrl: string;
    sex: string;
}	
```

/app/pet/pet.service.ts 
```ts
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { HttpClient } from '@angular/common/http';
import { Pet } from './pet'

@Injectable({
    providedIn: 'root'
})

export class PetService {
    private apiServerUrl = '';

    constructor(private http: HttpClient) {}

    public getPets(): Observable<Pet[]>{
        return this.http.get<Pet[]>(`${this.apiServerUrl}/pet/all`);
    }

    public addPet(pet: Pet): Observable<Pet>{
        return this.http.post<Pet>(`${this.apiServerUrl}/pet/add`, pet);
    }

    public updatePet(pet: Pet): Observable<Pet>{
        return this.http.put<Pet>(`${this.apiServerUrl}/pet/edit`, pet);
    }

    public deletePet(petId: number): Observable<void>{
        return this.http.delete<void>(`${this.apiServerUrl}/pet/delete/${petId}`);
    }

}
```
	
app.components.ts
```ts
import { Component, NgModule, OnInit } from '@angular/core';
import { Pet } from './pet/pet';
import { PetService } from './pet.service';
import { HttpErrorResponse } from '@angular/common/http';
import { NgForm } from '@angular/forms';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})

export class AppComponent implements OnInit {
  public pets: Pet[] = [];
  public editPet: Pet | undefined;
  public infoPet: Pet | undefined;

  constructor(private petService: PetService) {}

  ngOnInit() {
    this.getPets();
  }

  public getPets(): void{
    this.petService.getPets().subscribe(
    (response: Pet[]) =>{
      this.pets = response;
    },
    (error: HttpErrorResponse) =>{
      alert(error.message);
    }
    )
  }

  public onAddPet(addForm: NgForm): void{

    var a;
    if((a = document.getElementById('add-pet-form')) !== null){

    a.click();
    this.petService.addPet(addForm.value).subscribe(
      (response: Pet) => {
        console.log(response);
        this.getPets();
        addForm.reset();
      },
      (error: HttpErrorResponse) =>{
        alert(error);
        addForm.reset();
      }
    );
    }
  }

  public onUpdatePet(pet: Pet): void {
    this.petService.updatePet(pet).subscribe(
      (response: Pet) => {
        console.log(response);
        this.getPets();
      },
      (error: HttpErrorResponse) => {
        alert(error.message);
      }
    );
  }


  public onOpenModal(pet: Pet | null, mode: string) : void{
    const container = document.getElementById('main-container');
    console.log("container:" + container);
    const btn = document.createElement('button');
    btn.type = 'button';
    btn.style.display = 'none';
    btn.setAttribute('data-toggle', 'modal');

    if(mode === 'add'){
      btn.setAttribute('data-target', '#addPetModal');
    }

    if(mode === 'edit'){
      
      if(pet){
      this.editPet = pet;
      }

      btn.setAttribute('data-target', '#editPetModal');
    }

    if(mode === 'info'){
      if(pet){
        this.infoPet = pet;
        }
      btn.setAttribute('data-target', '#infoPetModal');
    }

    if(container !== null)
      container.appendChild(btn);

    btn.click();
  }
}
```

app.component.html
```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <a class="navbar-brand" style="color:white;">Adapost animale</a>
  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarColor02" aria-controls="navbarColor02" aria-expanded="false" aria-label="Toggle navigation">
  <span class="navbar-toggler-icon"></span>
  </button>
  <div class="collapse navbar-collapse" id="navbarColor02">
     <ul class="navbar-nav mr-auto">
        <li class="nav-item active">
           <a class="nav-link" (click)="onOpenModal(null, 'add')">Adaugare animal<span class="sr-only">(current)</span></a>
        </li>
     </ul>
  </div>
</nav>

<div class="container mt-3" id="main-container">
    <div class="row">
        <div  *ngFor="let pet of pets" class="col-md-3 col-xl-3">
            <div class="card m-b-30">
                <div class="card-body row" >
                      <a href=""><img src="{{pet?.imageUrl}}" alt="pet_image" class="card-img-top"></a>   
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item text-center"><i class="fa fa-paw float-center"></i> {{pet.name}} </li>
                </ul>
            
                <div class="card-body">
                  <div class="float-right btn-group btn-group-sm">
                        <a (click)="onOpenModal(pet, 'edit')" class="btn btn-primary tooltips" data-placement="top" data-toggle="tooltip" data-original-title="Edit"><i class="fa fa-pencil"></i> </a>
                        <a (click)="onOpenModal(pet, 'info')" class="btn btn-info tooltips" data-placement="top" data-toggle="tooltip" data-original-title="Info"><i class="fa fa-book"></i></a>
                  </div>
                </div>
              </div>
        </div>
    </div>
</div>

<!-- Add Pet Modal -->
<div class="modal fade" id="addPetModal" tabindex="-1" role="dialog" aria-labelledby="addPetModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="addPetModalLabel">Adaugare animal</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body">
        <form #addForm="ngForm" (ngSubmit)="onAddPet(addForm)">
          <div class="form-group">
            <label for="name">Nume</label>
            <input type="text" ngModel name="name" class="form-control" id="name" placeholder="Nume" required>
          </div>
          <div class="form-group">
            <label for="dateBirth">Data nastere</label>
            <input type="text" ngModel name="dateBirth" class="form-control" id="dateBirth" placeholder="Data nastere" required>
          </div>
          <div class="form-group">
            <label for="species">Specie</label>
            <input type="text" ngModel name="species" class="form-control" id="species" placeholder="Specie" required>
          </div>
          <div class="form-group">
            <label for="sex">Imagine</label>
            <input type="text" ngModel name="imageUrl" class="form-control" id="imageUrl" placeholder="Imagine">
          </div>
          <div class="form-group">
            <label for="sex">Sex</label>
            <input type="text" ngModel name="sex" class="form-control" id="sex" placeholder="Sex" required>
          </div>
           <div class="modal-footer">
             <button type="button" id="add-pet-form" class="btn btn-danger" data-dismiss="modal">Inchide</button>
             <button [disabled]="addForm.invalid" type="submit" class="btn btn-success" >Salveaza</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</div>


<!-- Edit Modal -->
<div class="modal fade" id="editPetModal" tabindex="-1" role="dialog" aria-labelledby="editPetModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
     <div class="modal-content">
        <div class="modal-header">
           <h5 class="modal-title" id="editPetModalLabel">Editare animal</h5>
           <button type="button" class="close" data-dismiss="modal" aria-label="Close">
           <span aria-hidden="true">&times;</span>
           </button>
        </div>
        <div class="modal-body">
           <form #editForm="ngForm">
              <div class="form-group">
                 <label for="name">Nume</label>
                 <input type="text" ngModel="{{editPet?.name}}" name="name" class="form-control" id="name" aria-describedby="emailHelp" placeholder="Nume">
              </div>
              <input type="hidden" ngModel="{{editPet?.id}}" name="id" class="form-control" id="id" placeholder="id">
              
              <div class="form-group">
                 <label for="dateBirth">Data naster</label>
                 <input type="text" ngModel="{{editPet?.dateBirth}}" name="dateBirth" class="form-control" id="dateBirth" placeholder="Data nastere">
              </div>
              <div class="form-group">
                 <label for="species">Specie</label>
                 <input type="text" ngModel="{{editPet?.species}}" name="species" class="form-control" id="species" placeholder="Specie">
              </div>
              <div class="form-group">
                 <label for="imageUrl">URL Imagine</label>
                 <input type="text" ngModel="{{editPet?.imageUrl}}" name="imageUrl" class="form-control" id="imageUrl" placeholder="URL Imagine">
              </div>
              <div class="form-group">
                 <label for="sex">Sex</label>
                 
                   <input type="text" ngModel name="sex" class="form-control" id="sex" placeholder="Sex" required>
                  <!--  <div id="sex">
                    <input type="radio" value="male" name="gender" required ngModel="{{editPet?.sex}}"> Male
                    <input type="radio" value="female" name="gender"required ngModel="{{editPet?.sex}}"> Female
                  </div>
                  -->
                 
              </div>
              <div class="modal-footer">
                 <button type="button" id="" data-dismiss="modal" class="btn btn-danger">Inchide</button>
                 <button (click)="onUpdatePet(editForm.value)" data-dismiss="modal" class="btn btn-success" >Salvare modificari</button>
              </div>
           </form>
        </div>
     </div>
  </div>
</div>

<!-- Info Pet  -->
<div class="modal fade" id="infoPetModal" tabindex="-1" role="dialog" aria-labelledby="infoPetModalLabel" aria-hidden="true">
  <div class="modal-dialog" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="infoPetModalLabel">Informatii animal</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>
      <div class="modal-body ">
      <form #editForm="ngForm">
        <div class="container d-flex justify-content-center">
          <div class="row">
        <div  >
            <div class="card">
                <div class="card-body row" >
                      <a href=""><img src="{{infoPet?.imageUrl}}" alt="pet_image" class="card-img-top"></a>   
                </div>
                <ul class="list-group list-group-flush">
                    <li class="list-group-item text-center"><i class="fa fa-paw float-center"></i> {{infoPet?.name}} </li>
                </ul>
            
                
                  <li class="list-group-item text-center">Id: {{infoPet?.id}}</li>
                  <li class="list-group-item text-center">Data nastere: {{infoPet?.dateBirth}}</li>
                  <li class="list-group-item text-center"> Sex: {{infoPet?.sex}}</li>
                
              </div>
        </div>
        
    </div>
    
        </div>
      </form>
      </div>
      <div class="modal-footer">
             <button type="button" id="add-employee-form" class="btn btn-primary" data-dismiss="modal">Inchide</button>
      </div>
    </div>
  </div>
</div>
```

Serverul local al paginilor web ruleaza pe localhost:4200 se face referinta catre localhost:8080 in envinronment.ts
```ts
export const environment = {
  production: false,
  apiBaseUrl: 'http://localhost:8080'
};
```


#### Rezultate

Afisarea intregului continut al tabelului
![list_all](https://user-images.githubusercontent.com/39569343/143071854-504d23c6-6abc-4882-a2fe-a16f1955716f.png)
	
Afisarea informatiilor
![info](https://user-images.githubusercontent.com/39569343/143071793-6d201eb0-616d-4214-a25e-e2a5de55af2a.png)
	
Afisarea dupa editare
![list_after_edit](https://user-images.githubusercontent.com/39569343/143071868-05bb324e-72b8-412b-bb5e-902a5bd1e271.png)
	
Editarea unui animal
![edit_pet](https://user-images.githubusercontent.com/39569343/143071768-73fc586b-a173-47c2-a90d-195eafd430d4.png)

Adaugare animal
![add](https://user-images.githubusercontent.com/39569343/143236887-b9a86546-e65d-4268-91fc-bc4ce637edc1.png)
	
Afisarea dupa adaugare
![list_after_add](https://user-images.githubusercontent.com/39569343/143236893-9f8f889e-6e28-4e50-a49c-441666dbccbc.png)
	
</details>

## _**Lab 7**_
<details>
  <summary>Apasti pentru a deschide laboratorul 7!</summary>
	
### **Exercitii:**
	
#### _*1. ORM - Object Relational Mapping ??? Database First.*_
	
Am folosit plugin-ul **POJO Generator** pentru a obtine clasa java din tabel.
	
Etape: Selectare **Databese** -> Tabelul: **Pet** ->  **POJO Generator** -> **Entity**.
	
![db](https://user-images.githubusercontent.com/39569343/145960715-eacd8ee4-be8e-45e6-b7e0-e2c0f6b3f94d.png)
	
Dupa parcurgerea etapelor de mai sus se genereaza un fisier Pet.java care contine clasa **Pet** asociata tabelui din baza de date.
	
![pet_class](https://user-images.githubusercontent.com/39569343/145966002-153953e8-4284-43f3-b384-c6781e1af767.png)

#### _*2. ORM - Object Relational Mapping ??? Code First.*_

**Pet.java** este modelul pe care se va genera tabelul in baza de date
	
```java
package ro.ppaw.shelter.model.pet;

import javax.persistence.*;
import java.io.Serializable;

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Pet implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;
    private String name;
    private String species;
    private String dateBirth;
    private Character sex;
    private String imageUrl;

    public Pet() {}

    public Pet(Long id, String name, String species, String dateBirth, String imageUrl, Character sex) {
        this.id = id;
        this.name = name;
        this.species = species;
        this.dateBirth = dateBirth;
        this.imageUrl = imageUrl;
        this.sex = sex;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSpecies() {
        return species;
    }

    public void setSpecies(String species) {
        this.species = species;
    }

    public String getDateBirth() {
        return dateBirth;
    }

    public void setDateBirth(String dateBirth) {
        this.dateBirth = dateBirth;
    }

    public Character getSex() {
        return sex;
    }

    public void setSex(Character sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Pet{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", species='" + species + '\'' +
                ", dateBirth='" + dateBirth + '\'' +
                ", sex=" + sex +
                '}';
    }
}
```
	
</details>

## _**Lab 8**_
<details>
  <summary>Apasti pentru a deschide laboratorul 8!</summary>
	
### **Exercitii:**
	
#### _*1. ORM - Object Relational Mapping ??? Database First.*_

Script.sql

```sql
-- creare tabel shelter
CREATE TABLE shelter(
    id  bigserial NOT NULL,
    address VARCHAR(255),
    image_url VARCHAR(255)
)

-- adaugare camp test in tabelul pet
ALTER TABLE pet ADD COLUMN test VARCHAR(200);
```

Tabelul nou creat **shelter** si campul adaugat **test**
	
![tales](https://user-images.githubusercontent.com/39569343/145984429-fd158202-2a62-4eae-8426-7c307d2a390e.png)
	
Clasa generata pe baza tabelului **pet**
	
![Pet](https://user-images.githubusercontent.com/39569343/145984430-d587c37b-e41b-4716-b0aa-f4543225231a.png)

Clasa generata pe baza tabelului **shelter** cu noul camp adaugat **test**
	
![Shelter](https://user-images.githubusercontent.com/39569343/145984419-03f82b5d-c492-4ccc-ac3f-eefe8048b732.png)
	
#### _*2. ORM - Object Relational Mapping ??? Code First.*_

a. In cazul migrarilor trebuie adaugat **Flyway** sau **Liquibase**  sau amandoua in fisierul pom.xml

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
	
# ...
	
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```
	
In fisierul **application.yaml** se specifica care se foloseste prin paramentru enable setat pe true sau false
	
```yaml
spring:
  liquibase:
    enabled: false
  datasource:
    username: postgres
    password: new123
    url: "jdbc:postgresql://localhost:5432/web_shelter"
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    hibernate:
      ddl-auto: none
    show-sql: true
  flyway:
    enabled: true

flyway:
  url: "jdbc:postgresql://localhost:5432"
  schemas: web_shelter
  user: marian
  password: new123
```

Pentru fiecare migrare se adauga un fisier in foldrul **migration** din calea **resources/db/migraion/** cu numele vesriuni urmat de doua underscor-uri si o descriere si cu extensia **.sql**, **.xml**, **.json**, **etc**.

![path](https://user-images.githubusercontent.com/39569343/145976376-8f404d20-4d8f-4769-a496-a395379d32f0.png)
	
**V1__PetTable.sql**
	
```sql
CREATE TABLE pet (
     id  bigserial NOT NULL,
     date_birth VARCHAR(255),
     image_url VARCHAR(255),
     name VARCHAR(255),
     sex CHAR(1),
     species VARCHAR(255), test VARCHAR(255),
     PRIMARY KEY (id)
)      
```

**V2__AlterPetTable.sql**

```slq
ALTER TABLE pet DROP COLUMN test;	
```

Outputul rularii si migrarea pentru **V1__PetTable.sql**
	
![output](https://user-images.githubusercontent.com/39569343/145980156-4bbbb5db-bbe4-456b-86b8-ace4cca27ed9.png)
	
![schema](https://user-images.githubusercontent.com/39569343/145980150-39d680f5-799c-443b-813c-ba6e033bdd8e.png)
	
Outputul rularii si migrarea pentru **V2__AlterPetTable.sql**
	
![output1](https://user-images.githubusercontent.com/39569343/145980162-ba6629cc-de9a-4cfd-825d-ef21979b2ce5.png)
	
![schema1](https://user-images.githubusercontent.com/39569343/145980155-92f7719d-c0db-48fe-ac96-3dd77910179d.png)

	
Continut tabel-ul de migrare - **flyway_schema_history**

![db_migrations](https://user-images.githubusercontent.com/39569343/145979388-1b57f500-a907-4c08-bbdc-36a4facee46f.png)

b. In cazul codului scris pentru generarea tabelului, la adaugarea unui camp in clasa la executarea programului acesta va fi introdus in tabel la fel si pentru eleminare. De asemenea trebuie sa se introduca/elimine din constructo campul specificat si metodele set/get al acestuia.

Adaugarea campului **test** in clasa Pet.java

```java
package ro.ppaw.shelter.model.pet;

import javax.persistence.*;
import java.io.Serializable;

@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Pet implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;
    private String name;
    private String species;
    private String dateBirth;
    private Character sex;
    private String imageUrl;
    private String test;

    public Pet() {}

    public Pet(Long id, String test, String name, String species, String dateBirth, String imageUrl, Character sex) {
        this.id = id;
        this.test = test;
        this.name = name;
        this.species = species;
        this.dateBirth = dateBirth;
        this.imageUrl = imageUrl;
        this.sex = sex;
    }

    public String getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(String imageUrl) {
        this.imageUrl = imageUrl;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getSpecies() {
        return species;
    }

    public void setSpecies(String species) {
        this.species = species;
    }

    public String getDateBirth() {
        return dateBirth;
    }

    public void setDateBirth(String dateBirth) {
        this.dateBirth = dateBirth;
    }

    public Character getSex() {
        return sex;
    }

    public void setSex(Character sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Pet{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", species='" + species + '\'' +
                ", dateBirth='" + dateBirth + '\'' +
                ", sex=" + sex +
                '}';
    }
}	
```

In timpul rularii programului se efectueza ALTER pe tabelul pet din baza de date de catre ORM-ul Hiberanate

![alter_table](https://user-images.githubusercontent.com/39569343/145970797-84c09f9b-96d2-4f07-b265-398ab7e21adf.png)

Tabelul inaintea modificarii

![pet_db_before](https://user-images.githubusercontent.com/39569343/145970469-0ac8c90b-ea63-499a-961c-bb52bb4f376e.png)
	
Tabelul dupa modificare
	
![table_after_add](https://user-images.githubusercontent.com/39569343/145970818-debaaa55-f2f6-4bc0-aa73-07526fc01648.png)
	
</details>

## _**Lab 9**_
<details>
  <summary>Apasti pentru a deschide laboratorul 9!</summary>
	
### **Exercitii:**
	
#### _*1. Dependency Injection.*_

Interfata - PetRepository.java
```java
package ro.ppaw.shelter.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import ro.ppaw.shelter.model.pet.Pet;

import java.util.Optional;

public interface PetRepository extends JpaRepository<Pet, Long> {

    void deletePetById(Long id);

    Optional<Pet> findPetById(Long id);
}
```

Dependency Injection este folosita prin intermediul anotarii **@Autowired**

#### _*2. Nivel Servicii (sau Business Layer).*_

Serviciul - PetService.java
	
```java
package ro.ppaw.shelter.servicies;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import ro.ppaw.shelter.exception.UserNotFoundException;
import ro.ppaw.shelter.model.pet.Pet;
import ro.ppaw.shelter.repository.PetRepository;

import javax.transaction.Transactional;
import java.util.List;

@Service
@Transactional
public class PetService {
    private final PetRepository petRepo;

    @Autowired
    public PetService(PetRepository petRepo){
        this.petRepo = petRepo;
    }

    public Pet addPet(Pet pet){
        return petRepo.save(pet);
    }

    public List<Pet> findAllPets(){
        return petRepo.findAll();
    }

    public Pet updatePet(Pet pet){
        return petRepo.save(pet);
    }

    public void deletePet(Long id){
        petRepo.deletePetById(id);
    }

    public Pet findPetById(Long id){
        return petRepo.findPetById(id).orElseThrow(() -> new UserNotFoundException("Pet with id " + id + " not found!"));
    }
}
```
	
</details>

## _**Lab 10**_
<details>
  <summary>Apasti pentru a deschide laboratorul 10</summary>
	
### **Exercitii:**
	
#### _*1. Stergere Fizica.*_
Pentru a sterge fizic fara pierderea datelor, se creeaza un tabel de tip istoric in care se va insera entitate care v-a fi eliminata din celelate tabele.
	
#### _*2. Stergere Logica.*_
Pentru stergerea logica adaugam in clasa (entitatea care reprezinta tabelul din baza de date) un camp de tip boolean. In cazul in care o inregistrare se v-a dori sa fie stearsa atunci valoarea acestui camo v-a fi modificata. Cand se vor extrage datele se v-a tine cont de acest camp acest operatie se realiza prin anotarile **@SQLDelete** si **@Where**.

Pet.java - adugarea camului **deleted**
	
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
@SQLDelete(sql = "UPDATE pet SET deleted = true WHERE id=?")
@Where(clause = "deleted=false")
public class Pet implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;
    private String name;
    private String species;
    private String dateBirth;
    private Character sex;
    private String imageUrl;
    private boolean deleted = Boolean.FALSE;

    @ManyToOne
    @JoinColumn(
            nullable = false,
            name = "shelter_id"
    )
    private Shelter shelter;

    public Pet() {}
}
	
```
	
Shelter.java - adugarea camului **deleted**
	
```java
@Entity
@SQLDelete(sql = "UPDATE shelter SET deleted = true WHERE id=?")
@Where(clause = "deleted=false")
public class Shelter implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;

    private String name;
    private String address;
    private String imageUrl;
    private Boolean deleted = Boolean.FALSE;

    public Shelter() {
    }
}
```

	
</details>

## _**Lab 11**_
<details>
  <summary>Apasti pentru a deschide laboratorul 11!</summary>
	
### **Exercitii:**
	
#### _*1. Cache.*_
Pentru a implementa Caching-ul trebui sa folosim urmatoarele anotari _**@EnableCaching**_ in clasa principla si _**@Cacheable("")**_ in serviciile a caror metode vor folosi cache-ul.

Clasa Principala - WebShelterApplication.java
```java
	@SpringBootApplication
@EnableCaching
public class WebShelterApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebShelterApplication.class, args);
    }

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.setAllowCredentials(true);
        corsConfiguration.setAllowedOrigins(Arrays.asList("http://localhost:4200"));
        corsConfiguration.setAllowedHeaders(Arrays.asList("Origin", "Access-Control-Allow-Origin", "Content-Type",
                "Accept", "Authorization", "Origin, Accept", "X-Requested-With",
                "Access-Control-Request-Method", "Access-Control-Request-Headers"));
        corsConfiguration.setExposedHeaders(Arrays.asList("Origin", "Content-Type", "Accept", "Authorization",
                "Access-Control-Allow-Origin", "Access-Control-Allow-Origin", "Access-Control-Allow-Credentials"));
        corsConfiguration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = new UrlBasedCorsConfigurationSource();
        urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(urlBasedCorsConfigurationSource);
    }
}
```
PetService.java	
```java
@Service
@Transactional
public class PetService {
    private final PetRepository petRepo;

    @Autowired
    public PetService(PetRepository petRepo){
        this.petRepo = petRepo;
    }

    public Pet addPet(Pet pet){
        return petRepo.save(pet);
    }

    @Cacheable("findAllPets")
    public List<Pet> findAllPets(){
        return petRepo.findAll();
    }

    public Pet updatePet(Pet pet){
        return petRepo.save(pet);
    }

    public void deletePet(Long id){
        petRepo.deletePetById(id);
    }

    @Cacheable("findPetsByShelterId")
    public List<Pet> findPetsByShelterId(Long id){
        return petRepo.getPetsByShelterId(id).orElseThrow(() -> new UserNotFoundException("Shelter with id: " + id + " doesn't have pets!"));
    }

    @Cacheable("findPetById")
    public Pet findPetById(Long id){
        return petRepo.findPetById(id).orElseThrow(() -> new UserNotFoundException("Pet with id " + id + " not found!"));
    }
}	
```

ShelterService.java	
```java
@Service
@Transactional
public class ShelterService {
    private final ShelterRepository shelterRepository;

    @Autowired
    public ShelterService(ShelterRepository shelterRepository) {
        this.shelterRepository = shelterRepository;
    }

    public Shelter addShelter(Shelter shelter){
        return shelterRepository.save(shelter);
    }

    @Cacheable("getShelters")
    public List<Shelter> getShelters(){
        return  shelterRepository.findAll();
    }

    @Cacheable("getShelterById")
    public Shelter getShelterById(Long id){
        return shelterRepository.findShelterById(id).orElseThrow(() -> new UserNotFoundException("Shelter with id " + id + " not found!"));
    }

    public Shelter editShelterById(Shelter shelter){
        return shelterRepository.save(shelter);
    }


    public void delteShelter(Long id){
        shelterRepository.deleteShelterById(id);
    }

}	
```
</details>
