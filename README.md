# Automated System Testing with Selenium2


### Test cases

```java
import org.junit.*;
import org.junit.runners.MethodSorters;
import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.support.ui.ExpectedCondition;
import org.openqa.selenium.support.ui.WebDriverWait;

import java.util.List;

import static org.hamcrest.CoreMatchers.is;
import static org.hamcrest.MatcherAssert.assertThat;

/**
 * Created by ismailcam on 25/03/2017.
 */

@FixMethodOrder( MethodSorters.NAME_ASCENDING )
public class SeleniumTest
{
    private static final int wait = 4;
    private static WebDriver webDriver;

    @Before
    public void setUp() throws Exception
    {
        // Set property
        System.setProperty( "webdriver.chrome.driver",
                            "/Users/ismailcam/IdeaProjects/TesingWithSelenium2/drivers/chromedriver" );

        // Reset Database
        com.jayway.restassured.RestAssured.given().get( "http://localhost:3000/reset" );

        webDriver = new ChromeDriver();
        webDriver.get( "http://localhost:3000" );
    }

    @After
    public void tearDown() throws Exception
    {
        webDriver.quit();

        // Reset Database
        com.jayway.restassured.RestAssured.given().get( "http://localhost:3000/reset" );
    }

    @Test
    // Verify that data is loaded, and the DOM is constructed (Five rows in the table)
    public void test1() throws Exception
    {
        ( new WebDriverWait( webDriver, wait ) ).until( new ExpectedCondition<Boolean>()
        {
            public Boolean apply(WebDriver wd)
            {
                int tableSize = wd.findElement( By.tagName( "tbody" ) )
                                  .findElements( By.tagName( "tr" ) )
                                  .size();

                assertThat( tableSize, is( 5 ) );

                return true;
            }
        } );
    }

    @Test
    // Write 2002 in the filter text and verify that we only see two rows
    public void test2() throws Exception
    {
        webDriver.findElement( By.id( "filter" ) ).sendKeys( "2002" );

        int tableSize = webDriver.findElement( By.tagName( "tbody" ) )
                                 .findElements( By.tagName( "tr" ) )
                                 .size();

        assertThat( tableSize, is( 2 ) );
    }

    @Test
    // Clear the text in the filter text and verify that we have the original five rows
    public void test3() throws Exception
    {
        webDriver.findElement( By.id( "filter" ) ).clear();

        int tableSize = webDriver.findElement( By.tagName( "tbody" ) )
                                 .findElements( By.tagName( "tr" ) )
                                 .size();

        assertThat( tableSize, is( 5 ) );
    }

    @Test
    // Click the sort “button” for Year, and verify that the top row contains the car with id 938 and the last row the car with id = 940.
    public void test4() throws Exception
    {
        webDriver.findElement( By.id( "h_year" ) ).click();

        List<WebElement> rows = webDriver.findElement( By.tagName( "tbody" ) )
                                         .findElements( By.tagName( "tr" ) );

        String firstId = rows.get( 0 )
                             .findElements( By.tagName( "td" ) )
                             .get( 0 )
                             .getText();

        String lastId = rows.get( rows.size() - 1 )
                            .findElements( By.tagName( "td" ) )
                            .get( 0 )
                            .getText();

        assertThat( firstId, is( "938" ) );
        assertThat( lastId, is( "940" ) );
    }

    @Test
    // Press the edit button for the car with the id 938. Change the Description to "Cool car", and save changes.
    // Verify that the row for car with id 938 now contains "Cool car" in the Description column
    public void test5() throws Exception
    {
        List<WebElement> rows = webDriver.findElement( By.tagName( "tbody" ) )
                                         .findElements( By.tagName( "tr" ) );

        for( WebElement row : rows )
        {
            List<WebElement> columns = row.findElements( By.tagName( "td" ) );

            if( columns.get( 0 ).getText().equals( "938" ) )
            {
                columns.get( columns.size() - 1 )
                       .findElements( By.tagName( "a" ) )
                       .get( 0 )
                       .click();

                WebElement descField = webDriver.findElement( By.id( "description" ) );
                descField.clear();
                descField.sendKeys( "Cool car" );
                webDriver.findElement( By.id( "save" ) ).click();

                assertThat( columns.get( 5 ).getText(), is( "Cool car" ) );

                break;
            }
        }
    }

    @Test
    // Click the new “Car Button”, and click the “Save Car” button. Verify that we have an error message with the
    // text “All fields are required” and we still only have five rows in the all cars table.
    public void test6() throws Exception
    {
        webDriver.findElement( By.id( "new" ) ).click();
        webDriver.findElement( By.id( "save" ) ).click();

        String errorMessage = webDriver.findElement( By.id( "submiterr" ) ).getText();

        int tableSize = webDriver.findElement( By.tagName( "tbody" ) )
                                 .findElements( By.tagName( "tr" ) )
                                 .size();

        assertThat( errorMessage, is( "All fields are required" ) );
        assertThat( tableSize, is( 5 ) );
    }

    @Test
    /*
        Click the new Car Button, and add the following values for a new car
        Year: 2008
        Registered: 2002-5-5
        Make: Kia
        Model: Rio
        Description: As new
        Price: 31000
        Click “Save car”, and verify that the new car was added to the table with all cars .
     */
    public void test7() throws Exception
    {
        webDriver.findElement( By.id( "new" ) ).click();

        webDriver.findElement( By.id( "year" ) ).sendKeys( "2008" );
        webDriver.findElement( By.id( "registered" ) ).sendKeys( "2002-5-5" );
        webDriver.findElement( By.id( "make" ) ).sendKeys( "Kia" );
        webDriver.findElement( By.id( "model" ) ).sendKeys( "Rio" );
        webDriver.findElement( By.id( "description" ) ).sendKeys( "As new" );
        webDriver.findElement( By.id( "price" ) ).sendKeys( "31000" );

        webDriver.findElement( By.id( "save" ) ).click();

        int tableSize = webDriver.findElement( By.tagName( "tbody" ) )
                                 .findElements( By.tagName( "tr" ) )
                                 .size();

        assertThat( tableSize, is( 6 ) );
    }
}
```
